# 아이템 14. Comparable을 구현할지 고려하라
`Comparable` 인터페이스에는 `compareTo`라는 메서드가 단 하나만 존재한다.
```java
public interface Comparable<T> {
  int compareTo(T t);
}
```

이는 `Object`의 메서드는 아니지만 `equals`와 비슷하다.

다른 점은 `compareTo`는 단순 동치성 비교뿐 아니라 순서까지 비교할 수 있고 `generic`하다. 

즉, `Comparable`을 구현했다는 것은 인스턴스들에 순서가 있다는 뜻이고 해당 객체들의 배열은 `Arrays.sort()`를 통해 정렬이 가능하다.

`String`은 `Comparable`을 구현하고 있기 때문에 아래 코드는 명령줄 인수들을 알파벳 순으로 출력한다.
```java
public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```

사실상 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입이 `Comparable`을 구현했기 때문에 위와 같이 편리하게 사용할 수 있다.

**만약 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 `Comparable`을 구현하자.**

## `compareTo` 일반 규약
이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수, 같으면 0, 크면 양의 정수를 반환한다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면
`ClassCastException`을 던진다.

* 반사성
  - `x.compareTo(y) < 0` 라면 `y.compareTo(x) > 0` 이다.
  - `x.compareTo(y`)가 `Exception`을 발생시킨다면 `y.compareTo(x)` 또한 `Exception`을 발생시켜야 한다.

* 추이성
  - `x.compareTo(y) < 0`고 `y.compareTo(z) < 0`라면 `x.compareTo(z) < 0`이다.

* 대칭성
  - `x.compareTo(y)` == 0 라면 `x.compareTo(z)` 와 `y.compareTo(z)` 는 같아야한다.

* 권고(필수는 아니지만 지키면 좋음)
  - `(x.compareTo(y) == 0) == (x.equals(y))`
  - 위처럼 equals 메서드와 일관 되도록 작성하는 것이 좋다.
  - 만약 일관되지 않게 구현을 했다면 그렇다는 것을 명시하자

위 규약을 지키지 못하면 비교를 활용하는 클래스와 함께 사용하지 못한다.
* `TreeSet`
* `TreeMap`
* `Collections`
* `Arrays`

### 규약의 주의사항
`equals`와 같이 `반사성`, `대칭성`, `추이성`을 충족해야하므로 주의사항도 동일하다.

기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 `compareTo` 규약을 지킬 방법이 없는데, 이를 우회하는 방법은 확장하는 대신 독립된 클래스를 만들고 이 클래스에 원래
클래스의 인스턴스를 가리키는 필드를 두면 된다.

그 다음 내부 인스턴스를 반환하는 `view`메서드를 제공하자. 이 방식을 사용하면 외부 클래스에서 `compareTo`를 구현해 넣을 수 있다.(추상화의 이점은 포기해야함)

```java
public class Point implements Comparable<Point> {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public int compareTo(Point point) {
        int result = Integer.compare(x, point.x);
        if (result == 0) {
            return Integer.compare(y, point.y);
        }
        return result;
    }
}

// ------------------------------------------------

class ColorPoint implements Comparable<ColorPoint> {
    private Point point;
    private int color;

    public ColorPoint(Point point, int color) {
        this.point = point;
        this.color = color;
    }

    public Point asPoint() {
        return point;
    }

    @Override
    public int compareTo(ColorPoint colorPoint) {
        int result = point.compareTo(colorPoint.point);
        if (result == 0) {
            return Integer.compare(color, colorPoint.color);
        }
        return result;
    }
}
```

## `compareTo` 작성 요령
`Comparable`은 제네릭 인터페이스이므로 `compareTo`의 인수 타입은 컴파일 시점에 정해진다. 인수의 타입 자체가 잘못되었다면 컴파일 자체가 되지 않는다.

또한 `null`을 인수로 넣어 호출하면 `NullPointerException`을 던져야한다.

`compareTo`는 각 필드의 동치를 비교하는 것이 아니라 순서를 비교한다. 객체 참조 필드를 비교하려면 `compareTo`를 재귀적으로 호출한다.

`Comparable`을 구현하지 않은 필드나 표준이 아닌 순서로 비교하려면 `Comparator`를 사용하자. 이는 직접 구현하거나 제공되는 것 중 골라서 사용하면 된다.

## 객체 참조 필드가 하나
```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
  private final String s;
  public int compareTo(CaseInsensitiveString cis) {
      return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
  public static void main(String[] args) {
        Set<CaseInsensitiveString> s = new TreeSet<>();
        for (String arg : args)
            s.add(new CaseInsensitiveString(arg));
        System.out.println(s);
    }
}
```

제네릭을 사용해 `CaseInsensitiveString`은 `CaseInsensitiveString`하고만 비교할 수 있다는 뜻이다.

`java 7`부터는 박싱된 기본 타입 클래스들에 새로 추가된 정적 메소드인 `compare`를 사용하여 정수 기본 타입 필드를 비교할 수 있다.

## 기본 타입 필드가 여러개일 때
가장 핵심 필드부터 비교하여 비교 결과가 0이 아니라면 바로 반환하면 된다.
```java
public int compareTo(PhoneNumber pn) {
  int result = Short.compare(areaCode, pn.areaCode);
  if (result == 0)  {
    result = Short.compare(prefix, pn.prefix);
    if (result == 0)
      result = Short.compare(lineNum, pn.lineNum);
  }
  return result;
}
```
## 비교자 생성 메서드를 활용한 생성자
`java 8`에서는 `Comparator` 인터페이스가 비교자 생성 메서드와 함께 메서드 연쇄 방식으로 비교자를 생성할 수 있다.

이는 코드의 간결함을 얻을 수 있지만, 성능 저하가 존재한다.
```java
private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber pn) -> pn.areaCode)
                                                          .thenComparingInt(pn -> pn.prefix)
                                                          .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
  return COMPARATOR.compare(this, pn);
}
```
`comparingInt`는 객체 참조를 `int` 타입 키에 매핑하는 키 추출 함수를 인수로 받아, 해당 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메소드이다.

`thenComparingInt`는 `Comparator`의 인스턴스 메소드로 `int` 키 추출자 함수를 입력받아 다시 비교자를 반환한다.(추가 비교 진행)

### 객체 참조용 비교자 생성 메소드
`comparing`이라는 정적 메서드는 2개로 다중 정의되어 있다.
1. 키 추출자를 받아 그 키의 자연적 순서를 사용하는 것
2. 키 추출자 하나와 추출된 키를 비교할 비교자까지 총 2개의 인수를 받음. 

`thenComparing` 인스턴스 메소드는 3개로 다중 정의되어 있다.
1. 비교자 하나만 인수로 받아 그 비교자로 부차 순서를 정함.
2. 키 추출자를 인수로 받아 그 키의 자연적 순서로 보조 순서를 정함
3. 키 추출자 하나와 추출된 키를 비교할 비교자까지 총 2개의 인수를 받음.

값의 차를 기준으로 첫 번째 값이 두 번째 값보다 작으면 음수, 같으면 0, 크면 양수를 반환하는 `compareTo` or `compare`메소드를 해시코드를 사용한 예시이다.
```java
// 정적 compare 메소드 사용
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return Integer.compare(o1.hashCode(), o2.hashCode());
  }
}

// 비교자 생성 메서드 사용
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```

