# 아이템 17. 변경 가능성을 최소화하라
`불변 클래스` : 인스턴스 내부 값을 수정할 수 없는 클래스
- `String`, 기본타입의 박싱 클래스들, `BigInteger`, `BigDecimal` 등등

불변 클래스는 가변 클래스보다 설계, 구현, 사용이 쉽고 오류가 생질 여지가 적어 안전하다.

<br>

## 불변 클래스

### 불변 클래스 규칙
- 객체 상태를 변경하는 메서드(변경자)를 제공하지 않는다.(`setter`)
- 클래스를 확장할 수 없도록 한다.
- 모든 필드를 `final`로 선언한다.
    - 설계자의 의도를 명확하게 드러내는 방법
- 모든 필드를 `private`로 선언한다.
    - 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근할 수 없도록 해준다.
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
    - 생성자, 접근자, `readObject` 메서드 모두에서 방어적 복사 수행
   
책에서 제시하는 불변 복소수 클래스이다. 
```java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

여기서 주목해야할 점은 `plus`, `minus`, `times`, `dividedBy`와 같은 사칙연산 메서드들이 자신을 수정하는 것이 아닌 **새로운 `Complex` 인스턴스**를
만들어 반환하는 점이다.

`add` 같은 동사 대신 `plus`와 같은 전치사를 사용한다는 점도 자신의 상태를 변하게 하지 않는다는 점을 부각시킨 것이다.
- `BigInteger`와 `BigDecimal`은 이 명명 규칙을 지키지 않아 사용자들이 잘못 사용하는 경우가 있다.

불변 객체는 단순히 생성된 시점부터 파괴되는 시점까지 상태를 그대로 보존한다.

**모든 생성자가 불변식을 보장한다면** 사용자가 별다른 노력을 들이지 않아도 불변으로 사용할 수 있다.

<br>

### 불변 객체의 장점

#### 불변 객체는 근본적으로 `Thread-Safe`하다. 따라서 안심하고 공유할 수 있다.
- 이러한 특성 때문에 불변 클래스의 인스턴스를 만들었다면 최대한 재활용하기를 권장한다.

`public static final` 상수로 만들어 재활용할 수 있다!!
```java
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE  = new Complex(1, 0);
public static final Complex I    = new Complex(0, 1);
```

1장에서 언급된 `정적 팩토리 메소드`를 사용하면 해당 인스턴스를 공유하기 때문에 동일한 효과를 얻을 수 있다.
- 메모리 사용량 감소
- GC 비용 감소

불변 객체에서는 방어적 복사의 의미가 없어지기 때문에 `clone`이나 복사 생성자를 제공하지 않는 것이 좋다.
- `String`의 복사 생성자는 되도록 사용하지 말자

불변 객체끼리는 **내부 데이터를 공유**할 수 있다.
- `BigInteger`는 값의 부호는 `int`를 크기는 `int[]`를 사용하는데 크기가 같고 부호만 반대인 새로운 `BigInteger`를 생성하는 `negate()`는 원본과 크기를 공유한다.

**객체를 만들 때 다른 불변 객체들을 구성요소**로 사용하면 아무리 복잡한 구조라도 불변식을 유지하기 훨씬 수월하다.
- `Map`, `Set`의 원소로 불변 객체를 사용하는 경우

불변 객체는 **실패 원자성**을 제공한다.
- 상태가 절대 변하지 않음

<br>

### 불변 클래스의 단점
값이 다르면 반드시 독립된 객체로 만들어야한다.
- 백만 비트를 가진 `BigInteger`인스턴스와 딱 1비트만 다른 `BigInteger`를 만들고 싶다면 완전히 독립된 새로운 객체를 만들어야한다.(`flipbit()`)
- 반대로 `BigSet`는 가변이기 때문에 원하는 비트하나만 바꿔주는 메서드를 제공한다.

이를 위한 **해결책**이 있다.

객체를 완성하기까지의 단계에서 쓰일 **다단계 연산**을 예측하여 제공해주는 것이다.

이를 **가변 동반 클래스**라고 한다.
- `String`을 예시로 들자면 `StringBuilder`와 `StringBuffer`가 있다.
- `BigInteger`는 모듈러 지수 같은 다단계 연산 속도를 높여주는 가변 동반 클래스를 `package-private`으로 두고 있다.

### 불변 클래스를 만드는 설계 방법들
클래스가 불변임을 보장하기 위해 `final`로 선언하는 것보다 더 유연한 방법을 소개한다.

**모든 생성자를 `private` or `package-private`로 만들고 `public 정적 팩토리`를 제공하는 방법이다.**

위의 `Complex`를 이 방식으로 구현해보자.
```java
public class Complex {

    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
}
```
`public`, `protected` 생성자가 없기 때문에 이 클래스를 다른 패키지에서 확장하는 것은 불가능하다.

따라서 이 불변 객체는 사실상 `final`이다.

이 방식은 다수의 구현 클래스를 활용한 유연성을 제공하고 객체 캐싱을 추가해 성능을 끌어올릴 수도 있다.

> 책을 읽다가 객체 캐싱이 문득 궁금해져 `Wrapper`클래스의 내부를 까보았다. 나는 객체 캐싱이라는 개념이 단순히 `Integer`를 반복해서 사용한다고 할 때
> 리스트에 담아두고 반환해준다는 개념으로만 생각을 했는데 `Wrapper`클래스의 내부 클래스로 `xxxCache`라는 것이 존재한다는 것을 발견했다.
> 이것과 이 `private`한 `xxxCache`와 `public 정적 팩토리`인 `valueOf`를 사용해 객체 캐싱을 할 수 있다!!
![image](https://user-images.githubusercontent.com/60773356/139037840-0342019d-5496-4ac7-87ea-5cbd583fa920.png)
![image](https://user-images.githubusercontent.com/60773356/139038425-ce8bf498-b67e-4d3a-9bda-ed32b66add26.png)


만약 신뢰할 수 없는 하위 클래스의 인스턴스라고 확인되면, 해당 인수들을 가변이라 가정하고 방어적 복사를 수행하자!!

<br>

### 성능을 위한 완화
`어떤 메서드도 불변 객체의 상태 중 외부에 비치는 값을 변경할 수 없다.`

바꿔 말하면

`불변 객체의 상태 중 외부에 비치지 않는 값은 변경할 수 있다.`

정도로 해석할 수 있겠다.

계산 비용이 큰 값을 나중에 계산해 `final`이 아닌 필드에 캐시해놓아 성능을 끌어올리는 방식을 사용할 수 있다는 의미이다.

이 방식은 순전히 해당 객체가 불변이기 때문에 가능한 묘수이다.
- 이 방식의 예로써 `String`의 `hash`필드가 존재한다.


<br>

### 직렬화시 주의점
`Serializable`을 구현하는 불변 클래스의 내부에 가변 객체를 참조하는 필드가 있다면 `readObject`나 `readResolve` 메소드를 반드시 제공하거나
`ObjectOutputStream.writeUnshared`와 `ObjectInputStream.readUnshared` 메소드를 사용해야한다.

그렇지 않다면 공격자가 해당 클래스로부터 가변 인스턴스를 만들어낼 수 있다.

#### 모든 클래스를 불변으로 만들 순 없다. 불변으로 만들 수 없더라도 변경 가능성을 최소화하자!!

#### 합당한 이유가 없다면 모든 필드는 `private final`이어야 한다.

#### 재활용의 목적으로 상태를 다시 초기화하는 메소드도 금지!!






