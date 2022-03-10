# 아이템 45. 스트림은 주의해서 사용하라

대량의 데이터 처리 작업을 지원하고자 추가된 것이 `Stream`이다.

### Stream API의 핵심
- 데이터 원소의 유한 혹은 무한 시퀀스를 뜻함
- 스트림 파이프라인은 원소들로 수행하는 연산 단계를 표현하는 개념(중간 연산, 종단 연산)

> 무한 시퀀스: 무한으로 이어지는 시퀀스

```
primes().map(prime -> BigInteger.valueOf(2).pow(prime.intValueExact()).subtract(BigInteger.valueOf(1)))
        .filter(mersenne -> mersenne.isProbablePrime(50))
        .forEach(System.out::println);
```
- 2 다음 소수를 무한히 반환하는 무한 시퀀스 예제 코드


스트림 안의 데이터 원소는 객체 참조나 기본 타입 값(int, long, double) 세가지를 지원한다.

스트림 파이프라인은 스트림으로 시작해서 종단 연산으로 끝나야하며 중간 연산이 하나 이상 사이에 존재할 수 있다.

중간 연산은 한 스트림을 다른 스트림을 변환하는 역할을 하는데 이전과 다를 수도 같을 수도 있다.
- filter(Predicate predicate) : predicate 함수에 맞는 요소만 사용하도록 필터
- map(Function function) : 요소 각각의 function 적용
- flatMap(Function function) : 스트림의 스트림을 하나의 스트림으로 변환
- distinct() : 중복 요소 제거
- sort() : 기본 정렬
- sort(Comparator comparator) : comparator 함수를 이용하여 정렬
- skip(long n) : n개 만큼의 스트림 요소 건너뜀
- limit(long maxSize) : maxSize 갯수만큼만 출력

종단 연산은 중간 연산이 내놓은 스트림을 정렬하거나 특정 하나를 선택하거나 하는 등의 최후의 연산을 한다.
- forEach(Consumer consumer) : Stream의 요소를 순회
- count() : 스트림 내의 요소 수 반환
- max(Comparator comparator) : 스트림 내의 최대 값 반환
- min(Comparator comparator) : 스트림 내의 최소 값 반환
- allMatch(Predicate predicate) : 스트림 내에 모든 요소가 predicate 함수에 만족할 경우 true
- anyMatch(Predicate predicate) : 스트림 내에 하나의 요소라도 predicate 함수에 만족할 경우 true
- noneMatch(Predicate predicate) : 스트림 내에 모든 요소가 predicate 함수에 만족하지않는 경우 true
- sum() : 스트림 내의 요소의 합 (IntStream, LongStream, DoubleStream)
- average() : 스트림 내의 요소의 평균 (IntStream, LongStream, DoubleStream)

<br>

### 지연 평가
스트림 파이프 라인은 지연평가된다.

종단 연산이 없는 파이프라인은 어떤 연산도 수행되지 않는다.

따라서 종단 연산을 빼먹는 일은 없도록 하자

<br>

---

스트림 API는 메서드 체이닝을 지원하는 `Fluent API`이다.

기본적으로 스트림 파이프라인은 순차적으로 진행되며 병렬로 실행하기 위해선 `parallel` 메서드를 호출해주면 된다.

하지만, 스트림이 만능인 것은 아니다.

스트림을 제대로 사용하면 프로그램이 짧고 간결해지지만 잘못 사용한다면 유지보수도 힘들어진다.

```java
// 잘못 사용한 예시 - 과한 스트림 사용
public class StreamAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```

```java
// 적절한 사용
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

참고로 람다에서는 타입을 생략하기 때문에 매개변수의 이름을 명확히 해주어야한다!

<br>

### char 값을 처리할 때는 스트림을 삼가는 편이 낫다
`chars()`는 반환 값이 `IntStream`이기 때문에 잘못 처리될 가능성이 크다.


<br>

### 스트림과 반복문을 적절히 조절하자
스트림으로 바꾸는게 간으할지라도 코드의 가독성이나 유지보수, 성능 측면에서 손해를 볼 수 있다.

기존 코드를 스트림을 사용하도록 리팩토링한다면 새 코드가 더 나아보일때만 반영하라.

<br>

### 스트림으로 할 수 없는 일
- 지역변수를 읽고 수정할 수 없음
- `return`, `break`등으로 메서드나 반복문을 종료할 수 없음


<br>

### 스트림으로 수행하면 좋은 일
- 원소들을 일괄적으로 변환
- 원소들을 필터링
- 원소들을 하나의 연산으로 결합(더하기, 연결, 최솟값)
- 원소들을 컬렉션에 모음
- 원소들 중에 특정 조건을 만족하는 원소 찾기


<br>

### 스트림으로 가능은 하지만 어려운 일
- 데이터가 파이프라인의 여러 단계를 통과할 때 각 단계에서 이 값을 동시에 접근하는 일
- 스트림 파이프라인은 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃기 때문

