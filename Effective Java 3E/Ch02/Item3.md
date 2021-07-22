# 아이템 3 : private 생성자나 열거 타입으로 싱글톤임을 보증하라

`싱글톤(singleton)`이란 `인스턴스`를 오직 하나만 가지고 있는 `클래스`를 말한다. 함수 같은 `무상태(stateless)`객체 또는 본질적으로 유일한 시스템 컴포넌트를 싱글톤으로 설계한다.

싱글톤을 만드는 방식은 두가지가 있다. 두 방법 모두 생성자를 `private`으로 만들고 `public static` 멤버를 통해 유일한 인스턴스를 제공하는 것이다.

## final 필드
```java
// 코드 3-1 public static final 필드 방식의 싱글턴 (23쪽)
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```
`private` 생성자는 `public static final` 필드인 `Elvis.INSTANCE`를 초기화할 때 딱 한번만 호출된다. 이는 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다.

예외로 권한이 있는 클라이언트가 `리플렉션 API`인 `AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있지만, 생성자가 호출될 때 호출된 횟수를 카운팅하거나
flag를 이용해 예외를 던지는 식으로 막을 수 있다.

### 장점
코드가 명확하고 간단하다

## static 팩토리 메서드
```java
// 코드 3-2 정적 팩터리 방식의 싱글턴 (24쪽)
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.getInstance();
        elvis.leaveTheBuilding();
    }
}
```
`Elvis.getInstance`는 항상 같은 객체의 참조를 반환하므로 이 또한 인스턴스가 오직 하나뿐임이 보장된다.(마찬가지로 `리플렉션 API`를 통한 예외는 가능)

### 장점
API를 변경하지 않고도 싱글톤을 사용할지 안할지를 변경할 수 있다. 싱글톤으로 사용하다가 추후에 쓰레드당 새 인스턴스를 만들도록 클라이언트 코드의 변경없이 수정할 수 있다.
```java
public static Elvis getInstance() { return new Elvis(); } // 새로운 인스턴스 생성하도록 변경
```

또한 필요하다면 `Generic 싱글톤 팩토리`를 만들 수도 있고, static 팩토리 메소드를 `Supplier<Elvis>`에 대한 `메소드 레퍼런스`로 사용할 수도 있다.
이러한 **장점들이 굳이 필요하지 않다면 public 필드 방식**이 좋다.

----------------------------
## 직렬화(Serialization)
위에서 살펴본 두 방법 모두, 직렬화에 사용한다면 역직렬화 할 때 같은 타입의 인스턴스가 여러개 생길 수 있다. 그 문제를 해결하려면 모든 인스턴스 필드에 `transient`를 추가 (직렬화 하지 않겠다는 뜻) 하고 
`readResolve` 메소드를 다음과 같이 구현하면 된다. [객체 직렬화 API의 비밀 참고](https://www.oracle.com/technical-resources/articles/java/serializationapi.html)
```java
// 싱글톤임을 보장해주는 readResolve 메서드
 private Object readResolve() {
    //진짜 Elvis를 반환하고, 가짜 Elvis는 GC에 맡긴다.
        return INSTANCE;
    }
```
```java
// 해결책
public class Elvis implements Serializable {
    private static final transient Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }
}
```
역직렬화는 기본 생성자를 호출하지 않고 값을 복사해서 인스턴스를 반환하는데 그때 사용하는 것이 `readResolve()` 메서드이다. 
직렬화 -> 역직렬화를 수행하면 Constructor 메시지는 직렬화 할 때 한번 출력되고 역직렬화 때는 출력되지 않지만 객체값을 비교해보면 다른 객체로 나온다.
`transient` 는 필드의 상태값을 전송하지 않는다는 키워드인데 여기서는 INSTANCE 의 상태값인 `new Elvis()`를 의미한다. 
`readResolve` 에서 `INSTANCE` 를 반환하는게 새로운 인스턴스 생성을 방지하는것이다.

## Enum
public 필드 방식과 비슷하지만, 더 간결하고 `직렬화/역직렬화`에 대한 문제를 해결할 필요없이, `리플렉션` 공격에 대한 문제도 해결할 수 있다.
```java
// 열거 타입 방식의 싱글턴 - 바람직한 방법 (25쪽)
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("기다려 자기야, 지금 나갈께!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```
부자연스럽다고 생각할 수 있지만 **대부분 상황에서 싱글톤을 만드는 가장 좋은 방법**이다. 하지만 이 방법은 Enum말고 다른 상위 클래스를 상속한다면 사용할 수 없다.
(인터페이스를 구현할 수 있음)

----------------------------
스프링을 사용하면 `스프링MVC`는 `스프링 컨테이너`를 통해 기본적으로 `스프링 빈`을 `싱글톤`으로 생성하고 관리하는 것을 알고 있을 것이다.
이것이 현실적인 싱글톤 사용법이다.(스프링을 사용한다면)




