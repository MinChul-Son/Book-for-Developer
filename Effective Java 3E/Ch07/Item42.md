# 아이템 42. 익명 클래스보다는 람다를 사용하라

```java

@FunctionalInterface
public interface adder {

    int add(int x, int y);
}
```

`java 8`부터 추상 메서드가 단 하나만 존재하는 인터페이스는 특별한 대우를 받게 되었다.

함수형 인터페이스라 부르는 인터페이스들의 인스턴스를 람다를 사용해 만들 수 있다.

**람다는 매우 간결하며 어떤 동작을 하는지 코드가 명확해진다.**

<br>

### 타입 추론
```
Collections.sort(words,
            (s1, s2) -> Integer.compare(s1.length(), s2.length());    
)
```
위 코드에서 람다의 매개 변수 `s1`과 `s2`의 반환 타입은 각각 `String`, `int`이지만 코드에서는 명시되어있지 않다.

이는 컴파일러가 문맥을 통해 타입 추론을 해준 것이기 때문에 타입을 명시해야 코드가 더 명확할 때를 제외하고는 람다의 모든 매개변수 타입은 생략하자!!

<br>

`NextStep`의 `TDD` 과정에서 문자열 계산기를 구현할 때 사용한 방식인데 책에서도 소개하고 있다.

```java
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

<br>

### 람다는 무조건 좋을까?
**람다는 이름이 없고 문서화를 할 수 없기 때문에 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수 가 많아지면 람다를 쓰지 말아야한다.**

람다는 한 줄일 때 가장 좋고, 길어도 세 줄 안에 끝내는 게 좋다.

람다가 길거나 읽기 어렵다면 간단히 줄여보거나 람다를 쓰지 않는 쪽으로 리팩토링 해보자!

추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으므로 익명 클래스를 써야 한다.

람다는 자기 자신을 참조할 수 없다.
> 람다에서 `this`는 바깥 인스턴스를 가리킨다.

따라서, 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다.

만약, 직렬화해야만 하는 함수 객체가 있다면 `private` 정적 중첩 클래스의 인스턴스를 사용하고 **람다나 익명 클래스를 직렬화하지 말자!**
