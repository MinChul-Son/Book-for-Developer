# 아이템 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

`Enum` 타입이 임의의 인터페이스를 구현하게하면 확장성있게 사용할 수 있다.

이번 넥스트 스텝의 문자열 계산기 미션을 사용하며 비슷한 느낌으로 `Enum`을 사용했었다.

```java
public enum Operator {
    PLUS("+", Operator::add),
    MINUS("-", Operator::sub),
    DIVIDE("/", Operator::divide),
    MULTIPLE("*", Operator::multiple);

    private String sign;
    private ToIntBiFunction<Integer, Integer> operate;

    Operator(final String sign, final ToIntBiFunction<Integer, Integer> operate) {
        this.sign = sign;
        this.operate = operate;
    }

    public static Operator from(final String inputSign) {
        return Arrays.stream(Operator.values())
                .filter(operator -> operator.sign.equals(inputSign))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException(ERROR_MESSAGE_SIGN));
    }

    private static int add(final int number, final int operand) {
        return number + operand;
    }

    private static int sub(final int number, final int operand) {
        return number - operand;
    }

    private static int divide(final int number, final int operand) {
        if (operand == 0) {
            throw new ArithmeticException(ERROR_MESSAGE_DIVIDED_BY_ZERO);
        }
        return number / operand;
    }

    private static int multiple(final int number, final int operand) {
        return number * operand;
    }

    public int operate(final int number, final int operand) {
        return this.operate.applyAsInt(number, operand);
    }

}
```

함수형 인터페이스인 `ToIntBiFuntion`을 사용하여 연산을 위한 코드를 만들었는데 이번 아이템에서 전달하고자하는 의도와 비슷하다고 느껴진다.


책에서의 코드를 살펴보면,
```java
public interface Operation {
    double apply(double x, double y);
}
```

```java
public enum BasicOperation implements Operation {
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

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

`Enum` 타입인 `BasicOperation`은 확장할 수 없지만 인터페이스인 `Operation`은 확장가능하다.

또한, `Operation`은 `apply()` 하나 밖에 없는 인터페이스이기 때문에 `@FuntionalInterface`을 붙혀 람다를 사용할 수도 있다.

이 방식을 사용하면 **해당 `Enum` 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 `Enum` 타입의 인스턴스로 대체할 수 있다!**

### 문제점
`Enum` 타입끼리 구현을 상속할 수 없다.




