# 아이템 5 : 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
대부분의 클래스는 여러 자원에 의존한다. 책에서는 `SpellChecker`와 `Dictionary`를 예시로 사용하였다. `SpellChecker`가 `Dictionary`를 사용하기 때문에 의존한다고 표현한다.

## 부적절한 예시
### static 유틸 클래스
```java
// 부적절한 static 유틸리티 사용 예 - 유연하지 않고 테스트 할 수 없다.
public class SpellChecker {

    private static final Lexicon dictionary = new KoreanDicationry();

    private SpellChecker() {
        // 인스턴스 생성 방지
    }

    public static boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }


    public static List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }
}

interface Lexicon {}

class KoreanDicationry implements Lexicon {}
```

### 싱글톤
```java
// 부적절한 싱글톤 사용 예 - 유연하지 않고 테스트 할 수 없다.
public class SpellChecker {

    private final Lexicon dictionary = new KoreanDicationry();

    private SpellChecker() {
    }

    public static final SpellChecker INSTANCE = new SpellChecker() {
    };

    public boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }


    public List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }
}
```

두 방식 모두 사전을 단 하나만 사용한다고 가정한다는 점에서 별로 좋은 구현이라고 생각되지 않는다. 실제로 언어마다 맞춤법 검사기는 사용하는 사전이 각기 다르고, 테스트 코드를 위한 테스트용
사전을 사용하고 싶을 수도 있다.

**즉, 사용하는 자원에 따라 동작이 달라지는 클래스에는 static 유틸리티 클래스나 싱글톤 방식은 적절하지 않다.**

--------------------------------
## 해결책
**인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식을 사용하면 된다.**(스프링의 DI를 생각하면 이해가 빠를 것이다.)

```java
public class SpellChecker {

    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }
    
    public List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }
}

class Lexicon {}
```
불변을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있다. 의존 객체 주입은 `생성자`, `static 팩토리`, `빌더`에 모두 적용할 수 있다.

이 패턴의 변형으로, **생성자에 자원 팩토리를 전달**하는 방법도 있다. 자바 8에 추가된 `Supplier<T>` 인터페이스가 팩토리를 표현한 완벽한 예이다.
* **팩토리**란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다.
`Supplier<T>`를 입력으로 받는 메서드는 일반적으로 한정적 와일드카드 타입(bounded wildcard type)을 사용해 팩토리의 타입 매개변수를 제한해야한다.
```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```
이 방식을 사용하면 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩토리를 넘길 수 있다.

의존 객체 주입이 유연성과 테스트 용이성을 크게 개선해주지만, 의존성이 많은 큰 프로젝트인 경우에는 코드가 어지럽고 장황해질 수 있다. 하지만 이 문제는 `Dagger`, `Guice`, `Spring`과 같은
의존 객체 주입 프로엠워크를 사용하면 해결할 수 있다.

## 결론
**의존하는 자원에 따라 동작을 달리하는 클래스를 만들 때는 static 유틸 클래스나 싱글톤을 사용하지말고 의존성 주입(DI)를 사용하자!!!**(유연함, 재사용성, 테스트 용이성 향상)






