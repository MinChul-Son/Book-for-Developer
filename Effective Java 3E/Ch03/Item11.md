# 아이템 11 : equals를 재정의하려거든 hashCode도 재정의하라
**`equals`를 재정의한 클래스 모두에서 `hashCode`도 재정의해야한다.** 

그렇지 않으면 `hashCode` 일반 규약을 어기게되어 해당 클래스의 인스턴스를 `HashMap`이나 `HashSet`같은 컬렉션의 원소로 사용할 때 문제가 발생한다.

## hashcode 규약
* `equals` 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 `hashCode`메서드는 몇 번을 호출해도 일관되게 항상 값은 값을 반환해야한다. 
  - 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
* `equals(Object)`가 두 객체를 같다고 판단했다면, 두 객체의 `hashCode`는 똑같은 값을 반환해야한다.
* `equals(Object)`가 두 객체를 다르다고 판단했더라도, 두 객체의 `hashCode`가 서로 다른 값을 반환할 필요는 없다.
  - 단, 다른 객체에 대해서 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

**논리적으로 같은 객체는 같은 해시코드를 반환해야한다!!!**

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
m.get(new PhoneNumber(707, 867, 5309)); // null을 반환한다!
```

논리적으로 동치이지만 `Map`에서 `get`을 하게되면 위의 코드에서는 `null`이 반환된다. 두 객체가 서로 다른 `hashCode`를 반환하기 때문이다.

설사 두 인스턴스를 같은 버킷에 담아도 여전히 `null`을 반환한다. `HashMap`은 `hashCode`가 다른 `entry`끼리는 동치성 비교를 하지 않도록 최적화되어 있기 때문이다.

이를 위해선 `PhoneNumber`의 `hashCode`를 재정의해주어야한다.

## 좋은 hashCode 구현
좋은 `hashCode`는 다른 인스턴스에 다른 `hashCode`를 반환해야한다.(32비트 정수 범위에 균일하게 분배해야함)
```java
// 전형적인 hashCode
@Override 
public int hashCode() {
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
```
* 첫번째 핵심 필드의 `hashCode`를 `result`로 초기화한다.
* 다음 핵심 필드의 `hashCode`를 기존 `result`와 `31`을 곱한 값에 더한다.
* 참조 타입 필드이면서 이 클래스의 `equals`메서드가 이 필드의 `equals`를 재귀적으로 호출해 비교한다면, 이 필드의 `hashCode`를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면 이 필드의 표준형을 만들어 그 표준형의 `hashCode`를 호출한다.
```java
// 예시! Rectangle의 hashCode를 계산하려면 재귀적으로 호출되어 Point의 hashCode까지 내려간다.
public class Rectangle{
  private Line width;
  private Line height;
  
  @Override public boolean equals(Object o){
    if(o == this)
      return true;
    if(!(o instatnceof Rectangle))
      return false;
    Rectangle r = (Rectangle) o;
    return width.equals(r.width) && width.equals(r.height);
  }
  
  @Override public int hashCode(){
    int result = width.hashCode();
    result = 31*result + height.hashCode();
    return result;
  }
}

public class Line{
  private Point start;
  private Point end;
  
  @Override public boolean equals(Object o){
    if(o == this)
      return true;
    if(!(o instatnceof Line))
      return false;
    Line l = (Line) o;
    return start.equals(l.start) && end.equals(l.end);
  }
  
  @Override public int hashCode(){
    int result = start.hashCode();
    result = 31*result + end.hashCode();
    return result;
  }
}

public class Point{
  private int x;
  private int y;
  
  @Override public boolean equals(Object o){
    if(o == this)
      return true;
    if(!(o instatnceof Point))
      return false;
    Point l = (Point) o;
    return x.equals(p.start) && y.equals(p.end);
  }
  
  @Override public int hashCode(){
    int result = x.hashCode();
    result = 31*result + y.hashCode();
    return result;
  }
}

```


* 필드의 값이 `null`이면 0을 사용한다.
* 필드가 배열이라면 핵심 원소 각각을 별도의 필드처럼 다룬다. 위의 규칙을 재귀적으로 적용한다. 배열에 핵심 원소가 하나도 없다면 상수(0)을 사용. 모든 원소가 핵심 원소라면 `Arrays.hashCode`사용
* 이 과정을 모든 핵심 필드에 적용시키고 최종 `result`를 반환한다.

파생 필드는 `hashCode` 계산에서 제외해도 된다. **`equals` 비교에 사용되지 않은 필드는 반드시 제거해야한다**. 

곱하는 상수 값이 `31`인 이유는 홀수이면서 소수이기 대문이다. 위의 `hashCode`는 논리적 동치인 인스턴스들은 동일한 해시코드를 가지게된다.

## Object 클래스의 정적 메서드 hash
`Object` 클래스는 임의의 개수만큼 객체를 받아 `hashCode`를 계산해주는 `hash`메서드를 제공한다. 이를 사용하면 `hashCode`를 한줄로 줄일 수 있다.

하지만 이 함수는 `parameter`를 담기 위한 배열이 만들어지고 입력 중 기본 타입이 있다면 `박싱/언박싱`을 거쳐야하므로 속도는 더 느리다. 성능에 민감하지 않을 때만 사용하자.
```java
@Override 
public int hashCode() {
  return Objects.hash(lineNum, prefix, areaCode);
}
```

## hashCode의 Lazy Initialization
클래스가 불변이고 해시코드를 계산하는 비용이 크다면 매번 계산하는 것보다 캐싱하는 방식을 사용하는 것이 좋다. 해당 객체가 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 `hashCode`를
계산해두어야 한다.

하지만 해시의 키로 사용되지 않는 경우에는 `hashCode`가 처음 불릴 때 계산하는 `Lazy Initialization(지연 초기화)` 방식을 사용하는 것도 좋은 방법이다.

이 방식을 사용하려면 해당 클래스를 스레드 안정성까지 고려하여 구현해야한다.
```java
private int hashCode; // 자동으로 0으로 초기화된다.

@Override 
public int hashCode() {
  int result = hashCode;
  if (result == 0) {
    result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    hashCode = result;
  }
  return result;
}
```
스레드 안정성을 위해 `syncronized`나 내부 클래스를 사용하는 방식으로 지연 초기화를 사용하면 된다. 여러 스레드에서 동시에 접근해서 초기화하면 객체 여러개가 생길 수 있으므로!!
```java
// 예시
class SomeClass {
  private Resource resource = null;
  public Resource getResource() {
    if (resource == null) {
      synchronized(SomeClass.class){
        if (resource == null)
          resource = new Resource();
      }
    }
    return resource;
  }
}
```


## hashCode를 계산할 때 핵심 필드를 생략하지마라
**성능을 높이기위한 목적으로 `hashCode`를 계산할 때 핵심 필드를 생략해서는 안된다!** 

속도는 빨라질 수 있어도 해시 품질이 나빠져 해시테이블의 성능이 심각하게 저하된다. 해시 테이블의 속도가 선형으로 느려질 것이다.

실제로 java 2 전의 `String`에는 이러한 문제가 있었다.

## `hashCode` 반환 값의 생성 규칙을 사용자에게 자세히 알려주지마라
`hashCode` 반환 값의 생성 규칙을 API 사용자에게 자세히 알려주게되면 클라이언트가 이 값에 의지하지 않게 되어 추후에 계산 방식을 바꿀 수도 있기 때문이다.

자바 라이브러리의 많은 클래스는 `hashCode`가 반환하는 정확한 값을 알려준다. 이는 명백한 실수!
