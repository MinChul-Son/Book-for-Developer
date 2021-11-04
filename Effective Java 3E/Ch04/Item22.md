# 아이템 22. 인터페이스는 타입을 정의하는 용도로만 사용하라
인터페이스는 자신을 구현한 **클래스의 인스턴스를 참조할 수 있는 타입 역할**을 한다.

즉, 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지 클라이언트에게 얘기해주는 것이다.
- 만약 `Comparable`을 구현한 클래스가 있다면 자신의 인스턴스로 순서를 매길 수 있다는 것을 의미한다.

#### 인터페이스는 오직 타입 역할의 용도로만 사용해야한다.

<br>

### 상수 인터페이스
```java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```

**상수 인터페이스**란 메소드 없이 `static final` 필드로만 가득 찬 인터페이스를 의미한다.

이는 **안티패턴**으로 인터페이스를 잘못 사용하는 예이다.

클래스 내부에서 사용하는 상수는 내부 구현에 해당한다. 따라서, 내부 구현을 인터페이스로 구현하는 것은 **외부로 노출시키는 것을 의미**한다.

클라이언트 코드가 상수들에 종속될 수도 있으며, 사용자에게 혼란을 주기도 한다.

만약 이후에 이 상수 인터페이스를 더 이상 쓰지 않게 되더라도 `바이너리 호환성` 때문에 기존 클래스에서 여전히 이를 구현해야할 수도 있다.

[바이너리 호환성](https://stackoverflow.com/questions/14973380/what-is-binary-compatibility-in-java)

<br>

### 해결책
**1. 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야한다.**
- `Integer`와 `Double`에 선언된 `MIN_VALUE`와 `MAX_VALUE`

**2. 열거(Enum) 타입으로 나타내기 적합한 상수라면 열거 타입으로 만들어라!**

**3. 인스턴스화할 수 없는 유틸리티 클래스에 담아 공개하라!**
```java
public class PhysicalConstants {
  private PhysicalConstants() { }  // 인스턴스화 방지

  // 아보가드로 수 (1/몰)
  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

  // 볼츠만 상수 (J/K)
  public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;

  // 전자 질량 (kg)
  public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```
- 유틸리티 클래스의 상수를 빈번히 사용한다면 `static import`와 함께 사용하여 더 깔끔하게 작성할 수 있다.



