# 아이템 55. 옵셔널 반환은 신중히 하라

`Optional`을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며 오류 가능성이 작다.

## Optional의 취지
`Optional`은 반환 값이 있을 수도 있고 없을 수도 있음을 클라이언트에게 명확히 알려준다.

## isPresent
안전밸브의 역할을 하는 메서드로 옵셔널에 값이 들어있다면 `true`를 비어있다면 `false`를 반환한다.

## 무조건 좋을까?
반환값을 `Optional`로 지정한다고 무조건 좋은 것은 아니다.

`Collection`, `Stream`, `Array`, `Optional`과 같은 컨테이너 타입은 `Optional`로 감싸선 안된다.

즉, `Optional<List<T>>`보다는 비어있는 `List<T>`가 좋다.


## Optional을 사용해야하는 상황
결과가 없을 수 있으며, 클라이언트가 이 상황에 대해 특별하게 처리를 해야한다면 `Optional`을 사용하자.

하지만 `Optional`도 엄연히 새로 할당하고 초기화하는 객체이고 내부의 값을 꺼내기 위해서 또 다른 메서드를 호출해야하는 단계를 거쳐야한다.

따라서, 성능이 중요하다면 `Optional`이 적절하지 않을 수 있다.


## 박싱된 기본 타입과 Optional
`OptionalInt`, `OptionalLong`, `OptionalDouble`과 같은 박싱된 기본 타입 전용 `Optional`이 존재하므로 박싱된 기본타입을 `Optional`에 담아 반환하지 말자.


#### Optional을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하지 말자.

