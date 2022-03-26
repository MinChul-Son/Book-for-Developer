# 아이템 52. 다중정의는 신중하게 사용하라

```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

`main`을 실행시켜보면 "그 외"라는 결과만 세번 출력된다.

이는 어느 메서드를 호출할지가 컴파일 타임에 결정되기 때문인데, `for`문안의 `c`는 항상 `Collection<?>`  타입이기 때문에 
정확히 일치하는 세번째 `classify`가 호출된다.

#### 재정의된 메서드는 동적으로 선택되고 다중정의한 메서드는 정적으로 선택된다.

```java
class Wine {
    String name() { return "포도주"; }
}

class SparklingWine extends Wine {
    @Override 
    String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
    @Override 
    String name() { return "샴페인"; }
}

public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
            new Wine(), new SparklingWine(), new Champagne());

        for (Wine wine : wineList)
            System.out.println(wine.name());
    }
}
```
여기서는 `main`을 실행시키면 "포도주", "발포성 포도주", "샴페인"이 출력된다.

관련해 `오브젝트 책의 12장 다형성`에 있는 내용을 참고하면 좋을 듯하다.

### 상속 관계에서의 동적 메서드 탐색
어떤 메서드를 호출할지가 런타임 시점에 결정된다.

1. 메시지를 수신한 객체는 먼저 자신을 생성한 클래스에 적합한 메서드가 존재하는지 검사한다. -> 만약 존재한다면 메서드 실행 후 종료
2. 메서드를 찾지 못했다면 부모 클래스에서 메서드 탐색을 진행한다. -> 적합한 메서드를 찾을 때까지 계층 구조를 따라 올라가며 탐색 진행
3. 최상위 클래스에서도 찾지 못했다면 예외를 발생시키며 종료한다.

즉, 책에서의 말처럼 **가장 하위에 정의된** 메서드가 실행되는 것이다.

### 다중정의
다중정의된 메서드 사이에서는 컴파일타임 타입에 의해 어떤 메서드가 실행될지 결정된다.

다중정의한 메서드는 개발자의 의도대로 동작하지 않을 가능성이 농후하기 때문에 헷갈릴 수 있는 코드를 작성하지 않는 것이 좋다.

**다중 정의로부터 발생할 수 있는 혼동을 피해야한다.**

### 다중정의 대신 메서드명을 다르게 지어주자
`ObjectOutputStream` 클래스의 `writeBoolean(boolean)`, `writeInt(int)`, `writeLong(long)`이 그 예시이다.

생성자는 무조건 다중정의가 되는데 이때는 정적 팩토리 메서드를 사용해서 문제를 해결해보자.

매개변수 중 하나 이상이 서로 근본적으로 다르다면 위와 같은 문제는 발생하지 않을 것이다.

이러한 조건이 충족된다면 매개변수들의 런타임 타입만으로 결정될 수 있다.

`ArrayList`의 생성자 중 `int`를 받는 것과 `Collection`을 받는 것은 어느 것이 호출될지 헷갈릴 일이 없다.

### List의 remove
`List<E>` 인터페이스는 `remove(Object)`와 `remove(int)`를 다중정의했기 때문에 특정 숫자를 제거하려했던 개발자의 의도와 달리
특정 인덱스의 숫자가 제거되는 상황이 발생할 수 있다.

이는 제네렉과 오토박싱의 등장으로 `Object`와 `int`가 근본적으로 다르지 않기 때문에 발생하는 문제이다.


#### 메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다.

### 기능이 같다면 괜찮다
다중정의된 메서드라도 서로의 기능이 동일하다면 신경쓰지 않아도 된다.

이러한 방식을 사용하는 것이 포워딩 방식이다.

```
public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence) sb);
}
```

### 잘못된 예시
`String`의 `valueOf(char[])`과 `valueOf(Object)`는 다중정의 되어있지만 같은 객체를 건내도 전혀 다른일을 수행하기 때문에

혼란을 불러올 수 있는 대표적인 사례이다.

