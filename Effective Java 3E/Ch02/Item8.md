# 아이템 8 : finalizer와 cleaner 사용을 피하라
`java`는 `finalizer`와 `cleaner` 두가지 객체 소멸자를 제공한다.

**`finalizer`는 예측할 수 없고, 상황에 따라 위험할 수 있어 대부분 불필요**하다. 오작동, 낮은 성능, 이식성 문제의 원인이 되기도 한다.

`java 9`부터는 `finalizer`가 `deprecated API`로 사용 자제되었고 대안으로 `cleaner`가 생겨났다. **`cleaner`는 별도의 쓰레드를 사용하여 `finalizer`보다는 덜 위험하지만
여전히 예측할 수 없고, 느리고 일반적으로 불필요**하다.

`java`에서는 비메모리 자원을 회수하기 위해 `try-with-resources`나 `try-finally`를 사용하여 해결한다.

## 단점 1
**`finalizer`와 `cleaner`는 언제 실행될지 알 수 없다**. 객체가 필요 없어진 시점에 즉시 `finalizer`와 `cleaner`가 실행되지 않을 수도 있다. 따라서, `finalizer`와 `cleaner`로는
제때 실행되어야 하는 작업은 불가능하다. 얼마나 신속하게 수행할지는 `Garbage Collerctor` 알고리즘에 달려있다.(즉, 구현마다 천차만별)

ex : 파일 리소스를 반납하는 작업을 그 안에서 처리한다면, 실제로 그 파일 리소스 처리가 언제 될지 알 수 없고, 자원 반납이 안되서 더이상 새로운 파일을 열 수 없는 상황이 발생할 수도 있다.

## 단점 2
**`finalizer`는 인스턴스의 자원 회수가 지연될 수 있다.** `finalizer` 쓰레드는 우선 순위가 낮아서 언제 실행될지 모르기 때문에 해당 쓰레드 안에 작업이 있다면 GC되지 않고 쌓여
결국에는 `OutOfMemoryError`가 발생할 수 있다.

`cleaner`는 별도의 쓰레드로 동작해 자신을 수행할 쓰레드를 제어할 수 있어 `finalizer`보다는 낫지만, 여전히 쓰레드는 백그라운드에서 동작하고 언제 처리될지를 알 수 없다.

## 단점 3
**`finalizer`와 `cleaner`는 수행 시점뿐 아니라 수행 여부조차 보장되지 못한다**. 즉, 해당 작업이 수행되지 못한 채 프로그램이 종료될 수도 있다.

따라서, 상태를 영구적으로 수정하는 작업에서는 절대 `finalizer`와 `cleaner`에 의존해서는 안된다.(DB와 같은 공유자원의 락 해제를 `finalizer`와 `cleaner`에 맡기면 전체 분산 시스템이 서서히
멈출 것이다.)

`System.gc`나 `System.runFinalization`에 속지마라. `finalizer`와 `cleaner`가 실행될 가능성을 높혀주는 것이지 실행을 보장해주지 않는다. 이를 보장하기 위해 만든
`System.runFinalizersOnExit`와 `Runtime.runFinalizersOnExit`는 수십년째 `deprecated` 상태이다.

또한 `finalizer`동작 중 발생한 예외는 무시된다.(경고조차 출력 X)

## 단점 4
**`finalizer`와 `cleaner`는 심각한 성능 문제도 동반된다.** `try-with-resource`로 자원을 반납하는데 걸리는 시간은 12ns였지만, `finalizer`는 550ns가 걸렸다.

## 단점 5
**`finalizer`를 사용한 클래스는 `finalizer 공격`에 노출되어 심각한 보안 문제가 발생할 수 있다.**

생성자나 직렬화 과정에서 예외가 발생하면 생성되다만 객체를 상속한 악의적인 하위 클래스의 `finalizer`가 수행될 수 있게된다. 이는 정적 필드에 자신의 참조를 할당해 `Garbage Collector`가
수집을 못하게 막을 수도, 예외가 발생한 상위 클래스의 메서드를 호출할 수도 있다.

생성자에서 예외가 발생해 존재하지 않아야하는 인스턴스인데, `finalizer`때문에 살아남은 것이다.

이를 방어하기 위해선 아무일도 하지 않는 `finalize`메서드를 만들고 `final`로 선언해서 상속하여 오버라이딩하는 것을 막아야한다.

## 대안은 없나?
자원 반납이 필요한 클래스는 `AutoCloseable`인터페이스를 구현하고 `try-with-resource`를 사용하거나 `try-finally`를 사용하여 `close`메서드를 명시적으로 호출하는 것이 정석이다.
```java
// try-with-resource 사용 예제
public class SampleResource implements AutoCloseable {
  @Override
  public void close() throws RuntimeException {
    // 자원 해제
    System.out.println("close");
  }
  public void hello() {
    System.out.println("hello");
  }
}
//--------------------------

public class SampleRunner {
  public static void main(String[] args) {
    try(SamepleResource sample = new SampleResource()) {
      sample.hello();
    }
  }
}
```
`sample`의 사용이 완료되면 자동으로 `close`되어 자원이 회수된다.

추가로 `close` 메서드에서 이 객체는 더 이상 유효하지 않음을 필드에 기록하고 객체가 닫힌 후에 `close`가 호출됐다면 `IllegalStateException`을 던진다.

## finalizer와 cleaner는 그럼 어디에 쓰는 거야?
### 1. 안전망 역할
클라이언트가 `close`메서드를 호출하지 않는 것에 대비한 안전망 역할이다. 늦게하더라도 아예 회수하지 않는 것보다는 낫다.
실제로 `FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor`, `java.sql.Connection`에는 안전망 역할의 `finalizer`가 있다.

### 2. 네이티브 피어
네이티브 피어란 일반 자바 객체가 네이트브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다. 이는 자바 객체가 아니므로 `Garbage Collecotr`가 존재를 모른다.
이 때 `finalizer`나 `cleaner`를 사용할 수 있다.


## cleaner 예제 코드
```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
    private static class State implements Runnable {
        int numJunkPiles; // Number of junk piles in this room : 정리할 자원

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출한다.
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
```
클라이언트가 `Room`의 `close()`메서드를 호출하면 `clean`이 호출되고 내부적으로 `run`을 호출한다.
만약 클라이언트가 `close`를 호출하지 않는다면(`Garbage Collector`가 회수할 때까지) `cleaner`가 `run`메서드를 호출해줄 것이다.

이 때 `cleaner`쓰레드(State)는 정리 대상인 인스턴스(Room)을 절대 참조해서는 안된다. 순환 참조가 생겨 `GC`의 대상이 되질 못하기 때문이다.

또한 `cleaner` 쓰레드를 만들 클래스는 `static`클래스여야한다. `static`이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 가지게되기 때문이다.

#### cleaner (Java 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자. 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.


## 참조
[try-with-resources](https://www.baeldung.com/java-try-with-resources)
[AutoCloseable](https://docs.oracle.com/javase/8/docs/api/java/lang/AutoCloseable.html)
[Cleaner](https://docs.oracle.com/javase/9/docs/api/java/lang/ref/Cleaner.html)



