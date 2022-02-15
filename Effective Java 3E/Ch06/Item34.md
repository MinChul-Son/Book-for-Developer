# 아이템 34. int 상수 대신 열거 타입을 사용하라

## 자바가 열거 타입을 지원하기 이전의 코드
```
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```
- `Type-Safe`를 보장할 방법이 없고 표현력도 좋지 않다.
- 평범한 상수를 나열한 것 뿐이기 때문에 깨지기 쉽다.
  - 상수의 값이 바뀌면 클라이언트도 다시 컴파일 해야함
- 정수 상수는 문자열로 출력하기가 까다롭다.
  - 상수가 몇 개인지 알기도 어렵다.
- 정수 대신 문자열 상수를 사용하는 변형 패턴인 `문자열 열거 패턴`은 더 나쁘다.

<br>

## Enum 타입
```java
public enum Fruit {
    APPLE, STRAWBERRY, MELON
}
```

이는 완전한 형태의 클래스이기 때문에 매우 **강력**하다.

`Enum`타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개한다.

밖에서 접근할 수 있는 생성자를 제공하지 않기 때문에 사실상 `final`이다.
- 생성자는 무조건 `private`이기 때문에 접근 못함

**따라서, `Enum` 타입으로 만들어진 인스턴스들은 하나씩만 존재함을 보장한다.**

<br>

### Type-Safe
컴파일타임 `Type-Safe`를 제공한다.

위의 예시코드에서 `Fruit` 타입을 메서드 파라미터로 받는다면 `APPLE`, `STRAWBERRY`, `MELON` 외의 타입이 제공될 때 컴파일 에러가 발생한다.

기본으로 제공하는 `toString`메서드는 출력하기 적합한 문자열을 반환해준다.
- 필요하다면 직접 `Override`도 가능하다.

<br>

### 메서드와 필드 추가
```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```
생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.

`Enum`은 근본적으로 불변이기 때문에 모든 필드는 `final`이어야한다.

상수를 하나 제거하더라도 제거한 상수를 참조만하지 않는다면 아무 영향이 없다.

제거한 상수를 참조한 곳에서는 컴파일 에러를 반환할 것이다.

<br>

### Enum의 생성 순서
1. `Enum`의 상수 변수(`primitive` 타입)
2. `Enum` 생성자
3. `Enum`의 `static` 변수

따라서, 생성자에서 자신의 인스턴스를 추가할 수 없다.
- 생성자가 호출될 시점에는 자신의 인스턴스가 생성되기 전이기 때문!!


```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    @Override public String toString() { return symbol; }

    public abstract double apply(double x, double y);

    // 코드 34-7 열거 타입용 fromString 메서드 구현하기 (216쪽)
    private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(
                    toMap(Object::toString, e -> e));

    // 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }

    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values())
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
    }
}
```
추상 메서드를 작성하고 이를 상수마다 구현하는 것또한 가능해진다.

<br>

### 언제 쓰면 좋을까?🤔
**필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자!!**





