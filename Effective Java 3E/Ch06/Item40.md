# 아이템 40. @Override 애노테이션을 일관되게 사용하라

`@Override`: 상위 타입의 메서드를 재정의했음을 의미하는 애노테이션

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```
이 클래스에서는 `Object`의 `equals`를 `override`가 아닌 `overload`하고 있다.

만약 `equals`에 `@Override` 애노테이션을 붙였다면 컴파일러는 반환타입이 올바르지 않다는 컴파일 에러를 발생시켰을 것이다.

**상위 클래스의 메서드를 재정의하려는 모든 메서드에 `@Override` 애노테이션을 달자!!**

예외적으로, 상위 클래스의 `abstract` 메서드를 재정의할 때는 `@Override`를 달지 않아도 괜찮다.

#### 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 @Override를 다는 것이 좋다.
