# RAW 타입은 사용하지말라

| 한글 용어         | 영문 용어                   | 예                                |
|---------------|-------------------------|----------------------------------|
| 매개변수화 타입      | Parameterized Type      | List<String>                     |
| 실제 타입 매개변수    | Actual Type Parameter   | String                           |
| 제네릭 타입        | Generic Type            | List<E>                          |
| 정규 타입 매개변수    | Formal Type Parameter   | E                                |
| 비한정적 와일드카드 타입 | Unbounded Wildcard Type | List<?>                          |
| Raw 타입        | Raw Type                | List                             |
| 한정적 타입 매개변수   | Bounded Type Parameter  | <E extends Number>               |
| 재귀적 타입 한정     | Recursive Type Bound    | <T extends Comparable<T>>        |
| 한정적 와일드카드 타입  | Bounded Wildcard Type   | List<? extends Number>           |
| 제네릭 메서드       | Generic Method          | static <E> List<E> asList(E[] a) |
| 타입 토큰         | Type Token              | String.class                     |

<br>

`List<String>`은 `List`의 원소가 `String`임을 뜻하는 매개변수화 타입이다.

여기서 경고하는 `Raw 타입`이란 타입 매개변수를 전혀 사용하지 않는 것을 말한다.
- `List`

이는 제네릭 이전의 코드들과의 호환성을 위해 존재하는 것!!

`Raw` 타입을 사용하게 되면 제네릭의 가장 큰 이점인 컴파일 시점의 타입 체크를 얻을 수 없다.

즉, 런타임 시점에 오류가 발생한다.(대참사)

우리가 `String` 타입만 담는 리스트를 만들 것이라고 가정해보자.

원소에 `String`만 존재할 것이라고 생각해 `get()`을 사용해 원소를 꺼내고 `String`타입으로 캐스팅을 시도할 때 만약 `Integer` 타입이었다면?

**`ClassCastException`이 발생한다!!**

<br>

반면에 매개변수화 타입을 사용한다면?
```
List<String> list = new ArrayList<>();

list.add(1); // 여기서부터 컴파일 오류 발생!!
```

다른 타입의 원소를 `add`하려는 순간 컴파일 오류가 발생한다.

**가장 좋은 에러는 컴파일 에러다!**

#### Raw 타입을 쓰면 제네릭이 안겨주는 안전성과 표현력을 모두 잃는다!

<br>

```java
public class Raw {
    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(42));
        String s = strings.get(0); // 컴파일러가 자동으로 형변환 코드를 넣어준다.
    }

    private static void unsafeAdd(List list, Object o) {
        list.add(o);
    }
}
```
이 코드는 컴파일은 되지만 `unsafeAdd`가 `Raw 타입`인 `List`를 사용하기 때문에 런타임 시점에 에러가 발생한다.

<br>

## 타입을 모를 때는 비한정적 와일드카드 타입을 사용하라
제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않다면 `?`를 사용하자

`Set<E>`의 비한정적 와일드카드 타입은 `Set<?>`이다.

Raw 타입은 어떤 타입이라도 넣을 수 있으니 타입 불변식을 훼손하기 쉽지만, 비한정적 와일드카드 타입은 `null`이외에 어떤 원소도 넣을 수 없다.

<br>

## class 리터널에는 Raw 타입을 써야한다
Java 명세에서 class 리터럴에 매개변수화 타입을 하지 못하게 하였다.
- 배열과 기본타입은 허용
- `List.class`, `String[].class`, `int.class`는 허용
- `List<String>.class`, `List<?>.class`는 허용되지 않음

<br>

## instanceof
제네릭은 런타임에 타입 정보가 없어지므로 `instanceof`는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없다.

또한, Raw 타입이든 비한정적 와일드카드 타입이든 `instanceof`는 동일하게 동작한다.
