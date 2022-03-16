# 아이템 47. 반환 타입으로는 스트림보다 컬렉션이 낫다

`Stream` 인터페이스는 `Iterable` 인터페이스가 정의한 추상 메서드를 모두 포함하며 `Iterable` 인터페이스가 정의한대로 동작한다.

하지만 `Iterable`을 상속하지 않았기 때문에 `for-each`로 반복할 수 없다.

이를 위해선 `Iterable`로 캐스팅하는 작업을 통해 우회할 수 있는데, 매우 좋지 않은 코드이다.
```
for (ProcessHandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator) {
    // 로직
}
```

`Stream`을 받아 `Iterable`을 반환하도록 어댑터 패턴을 적용해 처리할 수 도 있다.
```
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

for (ProcessHandle ph : iterableOf(ProcessHandle.allProcesses())) {
    // 로직
}
```

객체 시퀀스를 반환하는 메서드가 오직 `Stream Pipeline`에서만 쓰인다면 `Stream`을 반환하고, 반복문에서 쓰일 것이라면 `Iterable`을 반환하자.

---

## Collection
`Collection` 인터페이스는 `Iterable`의 하위 타입이고 `Stream` 메서드도 제공하므로 반복과 스트림을 동시에 지원한다.

**원소 시퀀스를 반환하는 공개 API의 반환 타입에는 `Collection`이나 그 하위 타입을 쓰는 게 일반적으로 최선이다.**

만약, 반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 방법을 생각해보자.

```java
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException("집합에 원소가 너무 많습니다(최대 30개).: " + s);
        return new AbstractList<Set<E>>() {
            @Override public int size() {
                return 1 << src.size();
            }

            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }

            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }

    public static void main(String[] args) {
        Set s = new HashSet(Arrays.asList(args));
        System.out.println(PowerSet.of(s));
    }
}
```

`AbstractCollection`을 활용해 `Collection` 구현체를 작성할 때는 `contains`와 `size`를 추가로 구현해야한다.

만약 이 두가지를 구현하기 어려운 상황이라면 `Collection`보다 `Stream`이나 `Iterable`을 반환하는 것이 좋다.(별도의 메서드도 두 방식 모두를 제공해도 좋음)

때로는 구현하기 더 쉬운 쪽을 선택하는 방법도 존재한다.

