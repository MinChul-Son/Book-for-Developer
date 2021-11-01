# 아이템 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

## 상속용 클래스는 재정의할 수 있는 메소드들을 내부적으로 어떻게 이용하는지 문서화해야한다.
클래스의 공개된 메소드에서 자신의 또 다른 메소드를 정의할 수 있기 때문에 메소드를 재정의하면 어떤 일이 발생하는지를 정확히 정리하여야한다.
- `public`, `protected` 메소드 중에 `final`이 아닌 모든 메소드는 재정의 가능

어떤 순서로 호출하는지, 호출 결과가 결과에 어떤 영향을 주는 지도 포함해야한다.

#### 재정의 가능 메소드를 호출할 수 있는 모든 상황을 문서화하자!(백그라운드 쓰레드나, 정적 초기화 과정에서의 호출 포함)

`API`문서 메소드 설명 끝에 있는 `Implementation Requirements`가 이 설명에 해당한다.
- `@implSpec`을 붙혀주면 `javadoc`이 생성해줌.

상속이 캡슐화를 깨뜨리기 때문에 클래스를 안전하게 상속할 수 있도록 하기 위해 내부 구현 방식을 설명해야한다.

<br>

## 클래스의 내부 동작 과정 중간에 끼어들 수 있는 `hook`(훅)을 잘 선별하여 `protected` 메소드 또는 (드물게) 필드로 공개해야할 수도 있다.
상속용 클래스에서 어떤 메소드, 필드를 `protected`로 공개해야할지는 예측을 해보고, 하위 클래스를 만들어 테스트해보는 것이 최선이자 유일하다.

`protected` 메소드는 내부 구현에 해당하므로 그 수는 최대한 적어야하고, 너무 적게 노출해 상속의 이점을 해치지 않아야한다.

#### 상속용 클래스는 배포 전 반드시 하위 클래스를 통해 검증을 진행해야함을 명심하자!

<br>

## 상속을 허용하는 클래스가 지켜야할 제약
### 1. 상속용 클래스의 생성자는 직간접적으로 재정의 가능 메소드를 호출하면 안된다.
상위 클래스의 생성자는 하위 클래스의 생성자보다 먼저 실행되기 때문에 하위 클래스에서 재정의한 메소드가 하위 클래스의 생성자보다 먼저 실행된다.

```java
public class Super {
    // 잘못된 예 - 생성자가 재정의 가능 메서드를 호출한다.
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
    }
}
```

```java
public final class Sub extends Super {
    // 초기화되지 않은 final 필드. 생성자에서 초기화한다.
    private final Instant instant;

    Sub() {
        instant = Instant.now();
    }

    // 재정의 가능 메서드. 상위 클래스의 생성자가 호출한다.
    @Override public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

위에서 언급한 것처럼 상위 클래스의 생성자가 먼저 호출되고 상위 클래스의 생성자는 재정의 가능 메소드를 호출하고 있기 때문에 문제가 발생한다.

![image](https://user-images.githubusercontent.com/60773356/139612515-ea371a93-2d22-4951-b461-0a4bd7a4d36b.png)
- 하위 클래스의 필드는 초기화되지 않았기 때문에 `null`이 출력됨
- `println`이기 때문에 `NullPointerException`이 발생하지 않았는데 만약 `instant`객체를 호출했다면 `NullPointerException`이 발생!!

#### `private`, `fianl`, `static` 메소드는 재정의가 불가능하므로 호출해도 상관없다.

<br>

### `Cloneable`과 `Serializable`
`Cloneable`과 `Serializable` 인터페이스는 상속용 클래스 설계를 더 어렵게 만든다.

`clone`과 `readObject` 메소드는 새로운 객체를 만들어내기 때문에 여기에 따르는 제약또한 생성자와 비슷하다는 점을 주의하다.

#### 즉, `clone`과 `readObject`는 직간접적으로 재정의 가능 메소드를 호출하면 안된다.

또한, `Serializable`을 구현한 상속용 클래스가 `readResolve`나 `writeReplace` 메소드를 갖는다면 이는 `private`이 아닌 `protected`로 선언해야한다.

`private`로 선언하면 하위 클래스에서 무시되기 때문이다.

<br>

### 상속용으로 설계하지 않은 클래스의 상속을 금지하라!
- 클래스를 `final`로 선언
- 모든 생성자를 `private`, `package-private`로 선언하고 정적 팩토리 만들기

<br>

`Set`, `List`, `Map`과 같이 핵심 기능을 정의한 인터페이스가 있고 이를 구현했다면 상속을 금지해도 개발하는 데 아무런 어려움이 없다.

표준 인터페이스를 구현하지 않았음에도 상속을 허용해야한다면 **재정의 가능 메소드를 클래스 내부에서 사용하지 않게하고 문서화하자**!

### private 도우미 메소드
```java
public class Super {
    public Super() {
        helpMethod();
    }

    public void overrideMe() {
        helpMethod();
    }

    //도우미 메소드
    private void helpMethod() {
        System.out.println("super method");
    }
}
public class Sub extends Super{
    private String str;
    public Sub() {
        str = "Sub String";
    }

    @Override
    public void overrideMe() {
        System.out.println(str);
    }


    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

재정의 가능 메소드는 자신의 코드를 `private` 도우미 메소드로 옮기고 이 도우미 메소드를 호출하도록한다.

재정의 가능 메소드를 호출하는 다른 코드들도 모두 이 도우미 메소드를 직접 호출하도록 수정하면 된다!

