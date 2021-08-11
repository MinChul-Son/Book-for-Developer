# 아이템 10 : `equals`는 일반 규약을 지켜 재정의하라

`equals`를 재정의하는 일을 간단한 문제로 판단할 수 있지만 실상은 그렇지 않다. 곳곳에 문제를 유발할 수 있는 요인이 도사리고 있다.

이를 위한 최선의 방법은 재정의하지 않는 것이다. 하지만 꼭 재정의가 필요한 상황도 있을 것이다. 이를 판단할 수 있어야한다.

## `equals`를 재정의하지 않는 것이 최선인 상황
### 1. 각 인스턴스가 본질적으로 고유할 때
값을 표현하는 것이 아닌 동작하는 개체를 표현하는 클래스
* ex : `Thread` ==> `Object`의 `equals`는 여기에 맞게 구현되어있다.

### 2. 인스턴스의 논리적 동치성을 검사할 일이 없을 때
`Object.equals`는 두 객체가 같은 인스턴스인지에 대한 `물리적 동치성`을 확인한다.
만약 `논리적 동치성`을 검사할 일이 없다면 재정의하지 않고 사용하면 된다.

### 3. 상위 클래스에서 재정의한 `equals`가 하위 클래스에도 들어맞을 때
대부분의 `Set`구현체는 `AbstractSet`이 구현한 `equals`를 상속받아 쓰고, `List`구현체는 `AbstractList`로부터, `Map`구현체는 `AbstractMap`으로부터 상속받아 사용한다.

### 4. 클래스가 `private`이거나 `package-private`이고 `equals` 메서드를 호출할 일이 없을 때
만약 더 철저히 `equals`의 사용을 막고싶다면 아래와 같이 작성하자.
```java
@Override
  public boolean equals(Object o) {
    throw new AssertionError(); // 호출 금지
  }
```
-----------------------------------

## `equals`를 재정의해야하는 상황
`물리적 동치성`이 아닌 값이 같은지를 판단하는 `논리적 동치성`을 확인해야한다면 `equals`를 재정의해야한다.
```java
  String foo1 = "민철";
  String foo2 = "민철";
  foo1.equals(foo2);
```
값 클래스가 대표적인 예시이다.(`String`, `Integer`) 위의 코드의 목적은 객체가 같은지보다는 값이 같은지를 검사하고 싶을 것이다.

사실 엄밀히 말하면 대부분의 `java`사용자들이 알고 있듯이 `String`은 불변이기 때문에 해당 코드에서 `foo1`과 `foo2`는 동일한 객체 인스턴스를 참조하고 있기 때문에 `물리적 동치성`을
확인하더라도 `true`가 반환될 것이다.

하지만 대부분의 경우에 `논리적 동치성`을 확인하고 싶어하고 또한 `논리적 동치성`을 검사하도록 재정의한다면 `Map`의 키와 `Set`의 원소로 사용할 수 있게된다.

예외적으로 값 클래스라하여도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스(`singleton`)라면 `equals`를 재정의하지 않아도된다. `Enum`이 바로 여기에 해당한다.

-----------------------------------------
## `equals` 메서드의 일반 규약
`equals` 메서드는 동치관계를 구현하며 다음을 만족한다.
* 반사성(reflexivity) : `null`이 아닌 모든 참조 x에 대해 `x.equals(x)`는 `true`이다.
* 대칭성(symmetry) : `null`이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`가 `true`이면 `y.equals(x)`도 `true`이다.
* 추이성(transitivity) : `null`이 아닌 모든 참조 값 x, y, z에 대해 `x.equals(y)`가 `true`이고 `y.equals(z)도 `true`면 `x.equals(z)`도 `true`이다.
* 일관성(consistency) : `null`이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)를 반복해서 호출하면 항상 `true`거나 항상 `false`를 반환한다.
* null 아님 : `null`이 아닌 모든 참조 값 x에 대해 `x.equals(null)`은 `false`다.

### 동치관계
집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는데 그 부분집합을 `동치류`(동치 클래스)라 한다. 

`equals`는 모든 원소가 같은 `동치류`에 속한 어떤 원소와도 서로 교환할 수 있어야하고 각 부분집합(`동치류`)사이의 관계를 `동치관계`라고 한다.

### 반사성
객체는 자기 자신과 같아야한다.

### 대칭성
두 객체는 서로에 대한 동치 여부에 동일하게 답해야한다.

대칭성을 위반하는 예시 코드이다.
```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배!
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
    
    public static void main(String[] args) {
      CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
      String s = "polish";
      
      cis.equals(s); // True!!
      s.equals(cis); // False!!
    }
```
위의 코드에서 `cis.equals(s)`는 `true`이지만 `s.equals(cis)`는 `false`가 반환된다. 이는 `String`의 `equals`는 `CaseInsensitiveString`을 모르기 때문이다. 대칭성을 위반하는 코드이다.

```java
List<CaseInsensitiveString> list = new ArrayList<>();
list.add(cis);
```
위와 같이 `ArrayList`에 `cis`를 추가한 후, list.contains(s)를 호출하면 현재 JDK버전(8 기준)에서는 `false`를 반환한다. `contains` 내부에서 `equals`를 사용하고 있다.

하지만 이는 버전이 바뀌거나 신규 JDK버전에서의 구현에 따라 어떤 값이 반환될지를 예측할 수 없다. 즉, **`equals`규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다.**

이를 해결하려면 `CaseInsensitiveString`의 `equals`를 `String`과도 연동하지 않아야한다.
```java
// 해결책
@Override 
public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }
```

### 추이성
추이성은 `x=y && y=z`라면 `x=z`라는 뜻이다. 상위 클래스를 상속받아 새로운 필드를 추가하는 상황을 생각해보자.
```java
// 부모 클래스
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
}

// 자식 클래스(색상 추가)
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    @Override 
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
```
`ColorPoint`에서 `equals`를 재정의하지 않는다면 `color`에 대한 비교를 할 수 없으므로 위와 같이 재정의하여 `ColorPoint`의 인스턴스이고 위치와 색이 동일하면 `true`를 반환하도록 구현했다.

이는 `대칭성`을 위반한다. `Point`의 `equals`를 사용해 `ColorPoint`와 비교한 것과 `ColorPoint`의 `equals`를 사용해 `Point`와 비교한 것은 결과가 다르다.

먼저, `Point`의 `equals`는 좌표만을 검사하기 때문에 색은 비교대상이 아니므로 좌표만 같다면 `true`를 반환한다. `ColorPoint`의 `equals`는 `ColorPoint` 인스턴스가 아니라면 무조건 `false`를 
반환하므로 `Point`에 어떤 값이 있건간에 `false`만 반환할 것이다.

그렇다면 `ColorPoint`의 `equals`를 변경하여 `Point`와 비교할 때는 색상을 무시하여 상위의 `equals`를 호출하면 문제가 해결될까?
```java
@Override 
public boolean equals(Object o) {
  if (!(o instanceof Point))
    return false;

  // o가 일반 Point면 색상을 무시하고 비교한다.
  if (!(o instanceof ColorPoint))
    return o.equals(this);

  // o가 ColorPoint면 색상까지 비교한다.
  return super.equals(o) && ((ColorPoint) o).color == color;
//    }
```
이는 대칭성은 만족시키지만, `추이성`을 위배하고 있다.
```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```
`p1.equals(p2)`와 `p2.equals(p3)는 `true`이지만 `p1.equals(p3)`는 `false`를 반환한다.

또한, 무한 루프에 빠질 위험이 존재한다. 책에서 글로 설명한 부분을 코드로 나타내보겠다.
```java

	@Override
    public boolean equals(Object o) {
        if(!(o instanceof Point))
            return false;

        if(!(o instanceof ColorPoint))
            return o.equals(this);

        return super.equals(o) && ((ColorPoint) o).color == color;
    }


	@Override
    public boolean equals(Object o) {
        if(!(o instanceof Point))
            return false;

        if(!(o instanceof SmellPoint))
            return o.equals(this);

        return super.equals(o) && ((SmellPoint) o).smell == smell;
    }

```
어딘가에서 `myColorPoint.equals(mySmellPoint)`를 호출하였다고 가정해보자. 

`SmellPoint`는 `Point`의 하위 클래스이므로 첫번째 if문을 통과한다. 두번째 if문에 걸려들어가 `mySmellPoint.equals(myColorPoint)`가 호출되고 서로가 서로를 호출하는 상황이 반복되며
결국 `StackOverflowError`가 발생할 것이다!!!!

사실 이는 OOP 언어의 동치관계에서 나타나는 근본적인 문제로 **구체 클래스를 확장해 새로운 값을 추가하면서 `equals` 규약을 만족시킬 방법은 존재하지 않는다.**

이 말은 `equals`안의 `instanceof`를 `getClass`로 바꾸면 규약도 지키고 값도 추가하며 구체 클래스를 상속할 수 있을 것이라 들릴 수 있다.
```java
// Point의 equals이다.
@Override 
public boolean equals(Object o) {
	if (o == null || o.getClass() != getClass())
		return false;
	Point p = (Point) o;
	return p.x == x && p.y == y;
}

// Point를 상속받는 하위 클래스
public class CounterPoint extends Point {
    private static final AtomicInteger counter =
            new AtomicInteger();

    public CounterPoint(int x, int y) {
        super(x, y);
        counter.incrementAndGet();
    }
    public static int numberCreated() { return counter.get(); }
}
```
이번 `Point`의 `equals`는 같은 구현 클래스의 객체와 비교할 때만 `true`를 반환한다. `Point`의 하위 클래스는 어디서든 `Point`로 활용이 가능해야하지만 `getClass`를 사용하면 그렇지 못한다. 이것은 우리가 흔히 아는 `SOLID`의 `LSP(리스코프 치환 원칙)`을 위반하고 있다. 이를 아래 코드로 알아보자.
```java
public class CounterPointTest {
    // 단위 원 안의 모든 점을 포함하도록 unitCircle을 초기화한다. 
    private static final Set<Point> unitCircle = Set.of(
            new Point( 1,  0), new Point( 0,  1),
            new Point(-1,  0), new Point( 0, -1));

    public static boolean onUnitCircle(Point p) {
        return unitCircle.contains(p);
    }

    public static void main(String[] args) {
        Point p1 = new Point(1,  0);
        Point p2 = new CounterPoint(1,  0);

        // true를 출력한다.
        System.out.println(onUnitCircle(p1));

        // false를 출력한다.
        System.out.println(onUnitCircle(p2));
    }
}
```
명확히 `LSP`를 위반하고 있음을 확인할수 있다. `contains`에서 내부적으로 `equals`를 사용하는데 `getClass`를 사용하였기 때문에 `false`가 반환된다. 만약 `instanceof`를 사용하였다면
`CounterPoint` 인스턴스를 넘겨주어도 정상 동작할 수 있다.

#### 상속 대신 컴포지션을 사용하라!
책에서 괜찮은 우회 방법으로 상속 대신 컴포지션을 사용하는 방법을 제시하고 있다.

```java
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    /**
     * 이 ColorPoint의 Point 뷰를 반환한다.
     */
    public Point asPoint() {
        return point;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}    
```

실제 자바 라이브러리에 구체 클래스를 확장해 값을 추가한 클래스가 종종 있다.

`java.sql.Timestamp`는 `java.util.Date`를 확장한 후 `nanoseconds`필드를 추가하였다. 결국 `Timestamp`의 `equals`는 대칭성을 위배하여 `Date`객체와 한 컬렉션에 넣거나 섞어 사용하게되면
의도치않은 방향으로 동작할 수 있는 문제가 있다.(둘을 명확히 분리해서 사용하면 문제없음)

참고로 **abstract 클래스의 하위 클래스라면 `equals` 규약을 지키면서 값을 추가할 수 있다. 위에서 다룬 내용은 상위 클래스를 직접 인스턴스로 만드는 것이 가능할 때의 이야기**이다.

### 일관성
두 객체가 같다면 수정되지 않는 한 영원히 같아야 한다는 뜻이다.(불변 객체로 만들지, 가변 객체로 만들지를 판단해야한다.)

**클래스가 불변이든 가변이든 `equals`의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.** `equals`는 항상 메모리에 존재하는 객체만을 사용한 결정적인 계산만 수행하여아한다.

예시로 `java.net.URL`의 `equals`는 주어진 `URL`과 매핑된 호스트의 `IP 주소`를 이용해 비교한다. 호스트 이름을 `IP 주소`로 바꾸려면 네트워크를 통해야하고, 결과가 항상 같다고 보장할 수 없다.

### null 아님
모든 객체가 `null`과 같지 않아야 한다는 뜻이다.

`instanceof`는 피연산자가 `null`이면 `false`를 반환하기 때문에 명시적으로 개발자가 `null`을 검사할 필요는 없다.

------------------------------------------
## 정리
### 순서
#### 1. == 연산자를 사용해 입력이 자신의 참조인지를 확인
`float`와 `double`은 `compare`을 통해 비교, 나머지는 `==`을 사용

#### 2. instanceof로 입력이 올바른 타입인지 확인
`null`도 정상 값으로 취급하는 참조 타입 필드도 있기 때문에 이런 필드는 정적 메서드인 `Object.equals`를 사용해 `NullPointerException`을 발생시키자.

#### 3. 입력을 올바른 타입으로 캐스팅(형변환)

#### 4. 파라미터로 넘어온 객체와 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 모두 검사

### 그 외
비교하기 매우 복잡한 필드를 가진 클래스라면 필드의 표준형을 저장해둔 후 표준형끼리 비교하면 경제적이다!(`final`을 통해 불변 필드로 표준형을 지정해둔다는 의미)

어떤 필드를 비교하냐에 따라 `equals` 성능이 좌우될 수 있다. 다를 가능성이 더 크거나 비교하는 비용이 싼 필드부터 비교하는 것이 성능에 도움이 된다.

동기화용 락(lock)필드 같이 객체의 논리적 상태와 관련없는 필드는 비교 대상이 아니다.

핵심 필드로부터 계산 가능한 파생 필드 또한 비교 대상이 아니다. 하지만 파생 필드를 비교하는 쪽이 더 빠른 경우에는 파생 필드를 비교하는 것이 좋다.(파생 필드가 객체 전체 상태를 대표할 때)

#### `equals`를 재정의할 땐 `hashcode`도 반드시 함께 재정의해라!

#### `equals`의 입력타입은 반드시 `Object`여야한다!


















