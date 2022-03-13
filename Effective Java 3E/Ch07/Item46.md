# 아이템 46. 스트림에서는 부작용 없는 함수를 사용하라

**계산을 일련의 변환으로 재구성하는 스트림**에서 각 변환 단계는 가능한 **이전 단계의 결과를 받아 처리하는 순수 함수**여야 한다.

여기서 순수 함수란 입력만이 결과에 영향을 주는 함수로써 다른 가변 상태를 참조하지 않고 함수 스스로도 다른 상태를 변경하지 않는 것을 의미한다.

```java
public class Freq {
    public static void main(String[] args) throws FileNotFoundException {
        File file = new File(args[0]);
        Map<String, Long> freq = new HashMap<>();
        try (Stream<String> words = new Scanner(file).tokens()) {
            words.forEach(word -> {
                freq.merge(word.toLowerCase(), 1L, Long::sum);
            });
        }
    }
}
```
- 스트림을 사용했지만 이점을 전혀 살리지 못했고 단순 반복적 코드!
- 오히려 가독성이 더 떨어지며 유지보수에도 좋지 않다.




```java
public class Freq {
    public static void main(String[] args) throws FileNotFoundException {
        File file = new File(args[0]);
        Map<String, Long> freq;
        try (Stream<String> words = new Scanner(file).tokens()) {
            freq = words
                    .collect(groupingBy(String::toLowerCase, counting()));
        }

        System.out.println(freq);
    }
}
```
- 이전 코드와 동일한 일을 하고 `Stream API`를 제대로 사용한 코드
- `forEach` 연산은 스트림 계산 결과를 보고할 때만 사용하고 계산할 때는 쓰지 말자
  - 스트림 계산 결과를 기존 컬렉션에 추가하는 등의 용도로는 사용할 수 있음

<br>

## Collector
축소 전략을 캡슐화한 블랙박스 객체

축소는 스트림 원소들을 객체 하나로 취합한다는 의미로써 `Collector`의 핵심 역할이다.

`Collector`를 사용하면 원소들을 손쉽게 컬렉션으로 모을 수 있다.

- `toList()`: 리스트를 반환
- `toSet()`: 집합을 반환
- `toCollection(collectionFactory)`: 사용자가 지정한 컬렉션 타입을 반환


```
List<String> topTen = freq.keySet().stream()
                .sorted(comparing(freq::get).reversed())
                .limit(10)
                .collect(toList());
```
- 가장 빈도가 높은 단어 10개를 찾아 리스트에 담는 코드

<br>

### toMap
`toMap(keyMapper, valueMapper)`: 스트림 원소를 키, 값에 매핑하는 함수를 인수로 받음
```
private static final Map<String, Operation> stringToEnum = 
    Stream.of(values()).collect(
        toMap(Object::toString, e -> e)
        )
```

<br>

`toMap(keyMapper, valueMapper, mergeFunction)`: 같은 키를 공유하는 값은 병합 함수를 사용해 기존 값과 합쳐질 수 있음
```
Map<Artist, Album> topHits = albums.collect(toMap(Album::artist, a -> a, maxBy(comparing(Album::sales))));
```
- 각 음악가의 베스트 앨범(가장 많이 팔린 앨범)을 맵으로 만드는 코드

또, 아래와 같이 충돌이 났을 때 더 늦게 들어온 값으로 바꾸는 역할도 가능하다.
```
toMap(keyMapper, valueMapper, (old, new) -> new);
```


<br>

`toMap(keyMapper, valueMapper, mergeFunction, mapFactory)`: 원하는 맵 구현체를 지정할 수 있음


<br>

### groupingBy
입력으로 분류 함수를 받아 출력으로 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다.
```
words.collect(groupingBy(word -> alphabetize(word));
```

만약 수집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면 다운 스트림 수집기도 명시해야한다.
```
Map<String, Long> freq = words.collect(groupingBy(String::toLowerCase, counting()));
```
- 카테고리에 속하는 원소의 개수로 매핑된 맵을 반환한다.


책에서 언급된 `partitioningBy`는 `groupingBy`와 매우 유사하지만 결과 없는 카테고리도 빈 리스트로 원소를 만든다는 차이가 있음


<br>

### minBy, maxBy
인수로 받은 비교자를 통해 최대값, 최소값을 찾아 반환한다.


<br>

### joining
이는 문자열 스트림에만 적용할 수 있다.

매개변수가 없는 `joining`은 단순히 원소들을 연결하는 수집기를 반환한다.

매개변수 하나를 받는 `joining`은 구분 문자를 매개변수로 받아 원소를 연결한다.

매개변수 세개를 받는 `joining`은 접두문자, 구분문자, 접미 문자를 받아 원소를 연결한다.
