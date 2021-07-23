# 아이템 4 : 인스턴스화를 막으려거든 private 생성자를 사용하라
단순히 정적 메서드와 정적 필드만을 담은 클래스를 만들고 있을 때가 있다.

## 예시
`java.lang.Math`, `java.util.Arrays` : 기본 타입 값이나 배열관련 메서드를 모아놓음

`java.util.Collections : 특정 인터페이스를 구현하는 객체를 생성해주는 `static`메서드를 모아놓음(자바 8부터는 `default` 메서드를 `interface`에 구현가능)

이러한 `static`멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 사용하려고 설계한 것이 아니다. 이런 클래스를 `abstract`로 만들어도 **상속받은 클래스로 인스턴스를 만들 수 있기 때문에**
인스턴스를 생성하는 것을 막을 수 없다. 또한, 생성자를 명시하지 않으면 `컴파일러`가 자동으로 매개변수를 받지 않는 `public` 기본 생성자를 만들어주기 때문에 이 경우에도 인스턴스를 생성할 수 있다.

인스턴스화를 막는 방법은 매우 간단하다. 명시된 생성자가 없을 경우에만 기본 생성자를 컴파일러가 만들기 때문에 **private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.**
```java
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
}
```
* 명시적 생성자가 `private`이기 때문에 **클래스 바깥에서는 접근할 수 없다**. `AssertionError`가 꼭 필요하지 않지만, 클래스 안에서 실수로라도 생성자를 호출하지 않게 해준다.
* 생성자를 제공하지만 쓸 수 없기 때문에 직관적이지 않기 때문에, 위의 코드처럼 적절한 주석을 추가해주자

이 방식은 **상속을 불가능하게 하는 효과**도 있다. 상속한 경우 명시적이든 묵시적이든 상위 클래스의 생성자를 호출하는데, 이 클래스의 생성자가 private으로 선언되어있기 때문에 호출을 할 수 없고
따라서 상속을 할 수 없다.
