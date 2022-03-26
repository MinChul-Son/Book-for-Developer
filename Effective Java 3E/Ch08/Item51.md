# 아이템 51. 메서드 시그니처를 신중히 설계하라

## 좋은 메서드 및 API 설계하기

### 메서드명을 신중히
이해할 수 있고, 다른 것들과의 일관성을 유지하라.

개발자 사이에서 널리 쓰이는 이름을 사용하고 긴 이름을 피하자.

애매할 때는 `Java API Guide`를 참조해보자.

### 편의 메서드를 너무 많이 만들지 마라
메서드가 너무 많으면 익히고 사용하고 문서화하고 테스트하고 유지보수하기 어려워진다.

이는 클래스, 인터페이스 모두 마찬가지!

클래스와 인터페이스는 자신이 맡은 기능을 완벽히 수행하는 메서드를 제공해야한다.

따라서, 확신이 서지 않는다면 만들지 말자.

### 매개변수 목록은 짧게 유지하라
4개 이하가 좋다.

이를 넘어간다면 기억하기 쉽지 않기 때문에 개발시에 문서를 참고하며 개발해야하는 상황이 발생할 수 있다.

같은 타입의 매개변수가 연달아 나오는 경우는 특히나 더 까다롭다.

### 매개변수 목록을 짧게 줄이는 방법
1. 여러 메서드로 쪼갠다.
2. 매개변수 여러 개를 묶어주는 도우미 클래스를 만든다.
3. 빌더 패턴을 메서드 호출에 응용한다.

### 매개변수 타입으로는 클래스보다 인터페이스가 좋다
`DIP`와 관련된 말이다.

구체화에 의존하지말고 추상화에 의존하자.

`Map` 인터페이스에 의존한다면 구현체인 `TreeMap`, `HashMap`을 넘겨도 아무런 문제없이 동작한다.

### boolean보다는 원소 2개짜리 열거 타입이 좋다.
> 메서드 이름상 `boolean`을 사용해야 의미가 명확해질 때를 제외하고

`Thermometer.newInstance(true)`라면 섭씨를 `Thermometer.newInstance(false)`라면 화씨를 사용하는 온도계 객체를 만드는 것보다

```java
public enum TemperatureScale {
    FAHRENHEIT, CELSIUS
}
```
열거 타입을 정의하고 `Thermometer.newInstance(TemperatureScale.CELSIUS)`와 같이 사용하는 것이 확장성에도 훨씬 좋고 가독성도 뛰어나다.
