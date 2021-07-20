# 아이템 2 : 생성자 매개변수가 많다면 빌더를 고려하라.
정적 팩터리와 생성자는 둘다 매개변수가 많을 때는 대응하기가 어렵다는 제약이 존재한다.

책에서는 예시로 `NutritionFacts`라는 영양정보를 표현하는 클래스를 만들어서 설명하고 있다.
------------------------------
## 1. 점층적 생성자 패턴(telescoping constructor pattern)
```java
// 코드 2-1 점층적 생성자 패턴 - 확장하기 어렵다! (14~15쪽)
public class NutritionFacts {
    private final int servingSize;  // (mL, 1회 제공량)     필수
    private final int servings;     // (회, 총 n회 제공량)  필수
    private final int calories;     // (1회 제공량당)       선택
    private final int fat;          // (g/1회 제공량)       선택
    private final int sodium;       // (mg/1회 제공량)      선택
    private final int carbohydrate; // (g/1회 제공량)       선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola =
                new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
    
}
```

이 방식은 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 필수 매개변수와 선택 매개변수 2개를 받는 생성자... 형태로 모든 매개변수를 받는 생성자까지
늘려나가는 방식이다. 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 사용자는 골라서 호출하면된다.

하지만 **매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.**

## 2. 자바빈즈 패턴(JavaBeans Pattern)
쉽게 설명하자면 이 방식은 아무 매개변수를 받지 않는 기본 생성자를 사용하여 인스턴스를 생성하고, `Setter`를 통해 필요한 필드만 설정한다.
```java
// 코드 2-2 자바빈즈 패턴 - 일관성이 깨지고, 불변으로 만들 수 없다. (16쪽)
public class NutritionFacts {
    // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
    private int servingSize  = -1; // 필수; 기본값 없음
    private int servings     = -1; // 필수; 기본값 없음
    private int calories     = 0;
    private int fat          = 0;
    private int sodium       = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)     { servings = val; }
    public void setCalories(int val)     { calories = val; }
    public void setFat(int val)          { fat = val; }
    public void setSodium(int val)       { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```
괜찮은 방법이라고 생각할 수 있지만 이것은 최종적인 인스턴스을 위해 `Setter`를 통한 필드들의 값 설정을 여러번 수행해야하고 최종적인 인스턴스이지 않은 상태에서
중간에 사용되는 경우 문제가 발생할 수 있다.(인스턴스가 완전히 생성되기 전까지 `일관성`이 무너짐)

또한 `Getter`와 `Setter`가 있어서 불변 클래스로 만들지 못하고(쓰레드 간 공유 가능한 상태) 쓰레드 안정성을 보장하려면 추가적으로 locking같은 작업이 필요하다.

책에서 객체를 수동으로 `Freezing`하고 그전에는 사용할 수 없도록 하는 방법도 있다고 하는데 이 방법은 다루기 어려워 실전에서는 거의 사용하지 않는다고 한다.
존재한다는 것만 알고 넘어가자.

## 3. 빌더 패턴(Builder Pattern)
1번의 안정성과 2번의 가독성을 모두 취할 수 있는 대안이 바로 빌더 패턴이다.

빌더 패턴은 클라이언트가 필요한 객체를 직접 만드는 것이 아니라, 필수 매개변수만으로 생성자(정적 팩토리)를 호출해 `Builder`객체를 얻는다.
그 다음 `Builder`객체가 제공하는 일종의 `Setter`메서드로 원하는 선택 매개변수들을 설정한다. 최종적으로 `build`라는 메서드를 호출해서 최종 객체를 생성한다.
동작 과정은 다음과 같다.
```java
// 코드 2-3 빌더 패턴 - 점층적 생성자 패턴과 자바빈즈 패턴의 장점만 취했다. (17~18쪽)
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
        { calories = val;      return this; }
        public Builder fat(int val)
        { fat = val;           return this; }
        public Builder sodium(int val)
        { sodium = val;        return this; }
        public Builder carbohydrate(int val)
        { carbohydrate = val;  return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) { // main : 실제 객체 생성 코드
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
}
```
main에서와 `Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();`와 같이 연쇄적으로 호출하는 것을 `플루언트 API(fluent API)` 또는 `메서드 연쇄(method chaining)`
이라고 한다.

빌더의 생성자나 메서드에서 유효성 확인을 할 수도 있고 여러 매개변수를 혼합해서 확인해야 하는 경우에는 `build`메서드를 호출하는 생성자에서 할 수 있다.
빌더에서 매개변수를 객체로 복사해온 뒤 확인하고 검증에 실패하면 `IllegalArgumentException`을 던지면된다.

**빌더 패턴은 계층적으로 설계된 클래스 구조에서 활용하기에 좋다.** 추상 빌더를 가지는 추상 클래스를 만들고 하위 클래스에서 추상 클래스를 상속받아 하위 클래스용 빌더도 추상 클래스 빌더를
상속받아 만들 수 있다.
```java
public abstract class Pizza {

    public enum Topping {
        HAM, MUSHROOM, ONION, PEEPER, SAUSAGE
    }

    final Set<Topping> toppings;

    abstract static class Builder<T extends  Builder<T>> { // `재귀적인 타입 매개변수`
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build(); // `Convariant 리턴 타입`을 위한 준비작업

        protected abstract T self(); // `self-type` 개념을 사용해서 메소드 체이닝이 가능케 함
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }

}
```
* `Objects.requireNonNull`은 단순히 `null` 값이라면 오류(`NullPointerException`)을 터트리는 메서드이다. 
```java
public class NyPizza extends Pizza {

    public enum Size {
        SMALL, MEDIUM, LARGE
    }

    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }


        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```
```java
public class Calzone extends Pizza {

    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauseInside = false;

        public Builder sauceInde() {
            sauseInside = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauseInside;
    }
}
```
`Pizza.Builder`는 `재귀적인 타입 매개변수`를 사용하여 `self`라는 메서드를 사용해 하위 클래스에서는 형변환(캐스팅)하지 않고도 메서드 연쇄를 사용할 수 있다.
```java
NyPizza nyPizza = new NyPizza.Builder(SMALL)
    .addTopping(Pizza.Topping.SAUSAGE)
    .addTopping(Pizza.Topping.ONION)
    .build();

Calzone calzone = new Calzone.Builder()
    .addTopping(Pizza.Topping.HAM)
    .sauceInde()
    .build();
```
각 하위 클래스 빌더의 `build`메서드는 해당 구체 하위 클래스를 반환하므로 캐스팅을 클라이언트가 하지 않아도 된다는 의미이다.
이 방식을 `Covariant 리턴 타이핑`이라고 한다.

가변 인자(varagrs) 매개변수를 여러개 사용할 수 있다는 소소한 정점도 있다. 메서드를 여러 번 호출하여 각 호출을 통해 전달된 매개변수를 하나의 필드로 모을 수도 있어 매우 **유연**하다.

단점으로는 객체를 만들기 전에 먼저 빌더를 만들어야 하는데 성능에 민감한 상황에서는 문제가 될 수도 있다.(하지만 빌더로 인해 성능 문제가 발생하는 경우는 굉장히 적다고 함)
또한, 생성자를 사용할 때보다 코드가 장황해진다.

**따라서, 빌더 패턴은 매개변수가 많거나 앞으로 늘어날 가능성이 있는 경우에 사용하면 좋다.**
항상, 정답은 없다. 어떤 것이 적절한 패턴인지를 판단하고 적용시키는 것이 개발자의 몫이라 생각한다.
[빌더 패턴 예시](https://github.com/greekZorba/java-design-patterns/tree/master/builder)

------------------------
## 추가
`Lombok`의 `@Builder` 애노테이션을 사용하면 빌더 패턴을 편리하게 사용할 수 있게 지원해준다.
```java
@Data
@Builder
public class User {
  private String name;
  private int age;
}
```
```java
// 빌더 사용
User user = User.builder()
                  .name("홍길동")
                  .age(19)
                  .build();
```
파라미터로 값을 지정해주지 않으면 각 타입에 맞게 `0/null/false`가 들어간다. 

`@Builder`가 제공하는 몇 가지 기능들을 살펴보자.
### 1. @Builder.Default
이 애노테이션은 값이 입력되지 않았을 때 기본 default값을 지정해줄 수 있다.
```java
@Data
@Builder
public class User {
  @Builder.Default
  private String name = "민철이";
  private int age;
}
```
```java
User user = User.builder()
                  .age(19)
                  .build();
```
위와 같이 `@Builder.Default`를 붙히고 값을 설정하면 파라미터가 오지 않았을 때 해당 값으로 default값이 설정된다.

### 2. @Singular
`@Singular`애노테이션은 단일 Object를 컬렉션에 추가할 수도, 컬렉션 자체를 추가할 수도 있게 해준다.
```java
@Data
@Builder
public class User {
  private String name;
  @Singluar
  private List<String> friends;
}
```
```java
User user = User.builder()
    .name("민철이")
    .friend("경우")
    .friend("승훈")
    .friend("우석")
    .friends(Arrays.asList("경우", "승훈", "우석"))
    .build();
```

## 3. builderMethodName 속성
이 속성으로 빌더 메서드의 이름을 지정하고 값을 검사하여 필수 파라미터가 입력되었는지를 검증할 수 있다.
```java
@Data
@Builder(builderMethodName = "userCheckBuilder")
public class User {
  private Long id;
  private String name;
  
  public static UserCheckBuilder builder(Long id) {
    if (id == null) {
        throw new IllegalArgumentException("필수 파라미터 누락");
    }
    return userCheckBuilder().id(id);
  }
}
```

이외에도 빌더 패턴을 도와주는 롬복의 애노테이션들이 존재하므로 필요 시에 찾아보면 좋을 듯하다.
[레퍼런스](https://projectlombok.org/features/Builder)

