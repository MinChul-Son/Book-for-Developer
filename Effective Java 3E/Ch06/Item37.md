# 아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라

```java
class Plant {

    enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }

    public static void main(String[] args) {
        Plant[] garden = {
            new Plant("바질", LifeCycle.ANNUAL),
            new Plant("캐러웨이", LifeCycle.BIENNIAL),
            new Plant("딜", LifeCycle.ANNUAL),
            new Plant("라벤더", LifeCycle.PERENNIAL),
            new Plant("파슬리", LifeCycle.BIENNIAL),
            new Plant("로즈마리", LifeCycle.PERENNIAL)
        };

        Set<Plant>[] plantsByLifeCycleArr =
            (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

        for (int i = 0; i < plantsByLifeCycleArr.length; i++)
            plantsByLifeCycleArr[i] = new HashSet<>();

        for (Plant p : garden)
            plantsByLifeCycleArr[p.lifeCycle.ordinal()].add(p);

        for (int i = 0; i < plantsByLifeCycleArr.length; i++) {
            System.out.printf("%s: %s%n",
                Plant.LifeCycle.values()[i], plantsByLifeCycleArr[i]);
        }
    }
}
```

`ordinal`로 배열을 인덱싱하는 코드이다 -> 매우 나쁜 코드

먼저, 배열은 제네릭과 호환되지 않으므로 비검사 형변환을 수행해야한다.

또한, 배열은 인덱스의 의미를 모르기때문에 출력 결과에 레이블을 추가해주어야 할 것이다.

<br>

## EnumMap
열거 타입을 `Key`로 사용하도록 설계한 `Map`구현체인 `EnumMap`을 사용하면 앞선 문제를 모두 해결할 수 있다.

```
    Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
            new EnumMap<>(Plant.LifeCycle.class);
            
    for (Plant.LifeCycle lc : Plant.LifeCycle.values())
        plantsByLifeCycle.put(lc, new HashSet<>());
            
    for (Plant p : garden)
        plantsByLifeCycle.get(p.lifeCycle).add(p);
    System.out.println(plantsByLifeCycle);
```

안전하지 않은 형변환을 쓰지 않고, `Key`가 `Enum` 이기 떄문에 그 자체로 출력용 문자열을 제공한다.

내부적으로 배열을 사용하기 때문에 뛰어난 성능을 가지고 있다 -> 이를 내부로 숨겨(캡슐화) 타입 안정성과 성능을 모두 얻어냄!

스트림을 사용해 코드를 더욱 간단하게 만들 수 있다.

```
System.out.println(Arrays.stream(garden)
            .collect(groupingBy(p -> p.lifeCycle)));
```

또한, 맵 구현체를 명시할 수도 있다.

```
System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle,
                () -> new EnumMap<>(LifeCycle.class), toSet())));
```
스트림을 사용한다면 해당 키에 매핑되는 값이 있을 때만 중첩 맵을 만든다. -> 스트림을 쓰지 않으면 모든 키 값에 대해 중첩 맵을 생성함

`EnumMap`안에 `EnumMap`을 집어넣어 매우 유연하게 사용할 수 있다.

```java
public enum Phase {
    SOLID, LIQUID, GAS;
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

> 상전이 맵: 이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵

> 쉽게 말해, 상태가 변화할 때 이전 상태를 대응시켜놓은 맵을 의미한다.

새로운 상태를 추가하기도 매우 쉽다.
```java
public enum Phase {
        SOLID, LIQUID, GAS, PLASMA;
        public enum Transition {
            MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
            BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
            SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
            IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);

        private final Phase from;
        private final Phase to;
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```
`PLASMA`와 `IONIZE`, `DEIONIZE`만 추가하였다.

`EnumMap`은 안전하고 명확하며 빠르다!
