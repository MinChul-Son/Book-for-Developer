# 아이템 49. 매개변수가 유효한지 검사하라

#### 입력 매개변수에 대한 제약은 반드시 문서화해야하며 메서드 몸체가 시작되기 전에 검사해야한다.

오류를 발생한 즉시 잡지 못하면 해당 오류를 감지하기 어려워지고 감지하더라도 발생지점을 찾기 어려워지기 때문이다.

<br>

### 매개변수 검사를 제대로 하지못하면 발생하는 문제
- 메서드가 수행되는 중간 모호한 예외를 던지며 실패할 수 있다.
- 메서드는 문제없이 수행되지만 잘못된 결과를 반환할 수 있다.
- 메서드는 문제없이 수행되지만 어떤 객체를 이상한 상태로 만들어놓아 미래의 어떤 시점에 메서드와 아무 관련없는 오류를 낼 수 있다.

> 매개변수 검사에 실패하면 **실패 원자성**을 어기는 결과를 낳을 수 있다.


### `public`, `protected` 메서드는 매개변수에 대한 예외를 문서화해야한다.
매개변수의 제약을 문서화하고  `@throws` 자바독 태그를 사용하여 그 제약을 어겼을 때 발생하는 예외도 함께 기술하자.

이를 통해 사용자가 제약을 지킬 가능성이 높아진다.


### `requireNonNull`을 사용하라
자바 7에 추가된 `requireNonNull`은 유연하고 사용하기 편하다.

이를 사용하면 `null`검사를 수동으로 하지 않아도 된다.

### 범위 검사 기능
자바 9에서는 `Object`에 범위 검사 기능인 `checkFromIndexSize`, `checkFromToIndex`, `checkIndex` 메서드가 추가되었다.


### 단언문
`public`이 아닌 메서드라면 단언문(assert)를 사용해 매개변수 유효성을 검증할 수 있다.

```java
private static void sort(long a[], int offset, int length){
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    
    // 정렬 로직
}
```
- 위 코드에서 `vm option`으로 `-ea`를 설정하고 실행했을 때 단언문의 값들이 하나라도 `false`라면 `AssertException`이 발생한다.


### 생성자
생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들이지지 않게 하는데 꼭 필요하다.


#### 단, 유효성 검사에 대한 비용이 지나치게 높거나 실용적이지 않다면 이 규칙을 굳이 따르지 않아도 된다.
- ex: 리스트를 정렬할 때 해당 리스트 안의 모든 원소가 정렬될 수 있는지를 검사하는 것
