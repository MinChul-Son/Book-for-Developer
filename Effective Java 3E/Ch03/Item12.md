# 아이템 12 : toString을 항상 재정의하라
`Object`의 기본 `toString`은 아래와 같다.
```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```
단순히 `클래스이름@해시코드`를 반환한다. 이는 `toString`을 사용하는 개발자가 원하는 것이 아니다. 좀 더 유익한 정보를 담는 형태로 재정의하는 것이 훨씬 좋을 것이다.

`toString`의 규약에서 **모든 하위 클래스에서 이 메서드를 재정의하라!** 고 말하고 있다.

`toString`을 잘 구현한다면 이를 사용한 시스템에서 디버깅이 쉬워진다. `toString`은 객체를 `println`, `printf`, `문자열 연결 연산자(+)`, `assert`구문에 넘길 때, 디버거가 객체를 출력할 때
자동으로 호출된다. 즉, `toString`을 직접 호출하지 않더라도 다른 곳에서 쓰인다.

`toString`은 **그 객체가 가진 주요 정보 모두를 반환하는 것이 좋다**. 하지만 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않다면 무리가 있다.

## toString 포맷의 문서화
`toString`을 구현할 때 반환값의 포맷을 문서화할지 정해야한다.

포맷을 명시하면 그 객체는 표준적이고 명확하며 사람이 읽을 수 있게 된다. 또한 그 값을 입출력에 사용해 사람이 읽을 수 있는 데이터 객체로 저장할 수도 있다.

만약 포맷을 명시한다면, 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리 메서드나 생성자를 함께 제공하면 좋다. 이는 자바의 많은 값 클래스가 사용하는 방식이다.(`BigInteger`, `BigDecimal`)

포맷을 한번 명시하면 평생 그 포맷에 얽히게 된다. 이 포맷에 맞춰 코드가 작성될 것이고 추후에 포맷이 변경된다면 이전에 작성하였던 코드는 엉망이 되기 때문이다. 이는 반대로 포맷을 명시하지 
않는다면 포맷 개선에 대한 유연성을 가지게된다.

**포맷 명시 여부와 관계없이 의도는 정확하게 밝혀야한다.** 

**`toString`이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자**.

## toString이 필요하지 않는 경우
정적 유틸리티 클래스(`private`로 인스턴스화를 막은 클래스)는 `toString`을 제공할 이유가 없다. 또한 열거타입(enum)도 `java`가 이미 완벽한 `toString`을 제공한다.

하지만 하위 클래스들이 공유할 문자열 표현이 존재하는 추상 클래스라면 `toString`을 재정의해야한다. 대부분의 컬렉션 수현체는 추상 컬렉션 클래스들의 `toString`을 상속하여 사용한다.




