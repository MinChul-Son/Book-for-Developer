# 아이템 6 : 불필요한 객체 생성을 피하라
똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 생성해놓고 재사용하는 편이 더 나을 때가 많다. **재사용은 빠르고 세련되다.**

책에서 이야기하는 절대 하지 말아야할 극단적 예시이다.
```java
String s = new String("안녕");
```
이는 실행될 때마다 새로운 `String` 인스턴스를 만들어낸다.
```java
String s = "안녕";
```
새로운 인스턴스를 만드는 대신 하나의 `String` 인스턴스를 사용하기 때문에 `JVM`안에서 이와 같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용한다.

## static 팩토리 메서드
이 책에서는 static 팩토리 메서드에 대한 내용이 계속해서 언급되고 있다. 생성자는 반드시 새로운 객체를 만들어야 하지만 static 팩토리 메서드는 그렇지 않다.

자바 9에서 `deprecated`된 `Boolean(String)` 대신 `Boolean.valueOf(String)` 같은 static 팩토리 메서드를 사용할 수 있다.

## 생성 비용이 비싼 객체
생성하는데 메모리나 시간이 오래걸리는 생성 비용이 비싼 객체를 반복적으로 만들어야한다면 캐싱하여 재사용하길 권한다.

책에서 예시로 제시한 정규 표현식 코드이다.
```java
static boolean isRomanNumeralSlow(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }

```
`String.matches`는 **정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않다.**

`String.matches`가 내부에서 만드는 정규표현식용 `Pattern` 인스턴스는 한 번 사용되고 바로 버려져 `GC`의 대상이된다. `Pattern`은 입력받은 정규표현식에 해당하는 
`유한 상태 머신(finite state machine)`을 만들기 때문에 생성비용이 높다.

`Pattern` 객체를 만들어놓고 재사용하는 것이 좋다.
```java
public class RomanNumber {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```
`isRomanNumeral`이 반복적으로 호출되는 상황에서 성능을 끌어올려줄 수 있지만, `isRomanNumeral` 메서드가 호출되지 않는다면 `ROAM`필드는 쓸데없이 만들어진 셈이 된다.
`지연 초기화(lazy initialization)`을 사용해 최적화할 수 있지만 추천하지 않는다. 지연 초기화는 코드를 복잡하게 만드는데 성능은 크게 개선되지 않는 경우가 많기 때문이다.

## 어댑터
**객체가 불변이라면 재사용하여도 안전함이 명확하다.** 하지만 상황에 따라 덜 명확하거나 직관에 반대되는 상황도 있다.

책에서는 어탭터를 예로 들었는데 어댑터는 `interface`를 통해서 뒷단 객체와 연결해주는 객체이기 때문에 여러개를 만들 필요가 없다. 즉, 제2의 `interface` 역할을 해주는 객체이다.

`Map`인터페이스가 제공하는 `keySet`은 `Map`이 뒤에 있는 `Set`인터페이스의 뷰(어댑터)를 제공한다. `keySet`을 호출할 때마다 새로운 객체가 생성될 것같지만 사실은 같은 객체를
리턴하기 때문에 리턴받은 `Set`타입의 객체를 변경하면 결국 그 뒤의 `Map`객체를 변경하게 되는 것이다.
```java
public class UsingKeySet {

    public static void main(String[] args) {
        Map<String, Integer> menu = new HashMap<>();
        menu.put("Burger", 8);
        menu.put("Pizza", 9);

        Set<String> names1 = menu.keySet();
        Set<String> names2 = menu.keySet();

        names1.remove("Burger");
        System.out.println(names2.size()); // 1
        System.out.println(menu.size()); // 1
    }
}
```

## 오토박싱
불필요한 객체를 생성하는 또 다른 예시로 `오토박싱`이 있다. 오토박싱은 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다.(박싱, 언박싱)

**오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 경계를 흐려주지만, 완전히 없애주는 것은 아니다.**
```java
public class AutoBoxingExample {

    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        Long sum = 0L;
        for (long i = 0 ; i <= Integer.MAX_VALUE ; i++) {
            sum += i;
        }
        System.out.println(sum);
        System.out.println(System.currentTimeMillis() - start);
    }
}
```
모든 양의 정수 총합을 구하는 코드이다. 이는 정확한 답을 도출하기는 하지만 문자하나 때문에 성능이 매우 나쁘다.

`sum`변수의 타입을 `Long`으로 만들었기 때문에 불필요한 `Long` 인스턴스가 "2의 31승"개 만큼 생성된다. (`long`타입의 `i`가 `Long`타입의 `sum`에 더해질 때마다)
`sum`을 `long`타입으로 바꾸는 것만으로도 10배정도의 성능차이가 발생한다.

**박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱을 하지 않도록 주의하자!**

**이를 `객체 생성은 비싸니 피해야한다`로 오해해서는 안된다**. 현재의 `JVM`은 상당히 최적화가 되어 가벼운 객체용을 다룰 때는 직접 만든 객체 풀보다 훨씬 빠르다.

특히, **방어적 복사(Defensive copying)를 해야 하는 경우에도 객체를 재사용하면 심각한 버그와 보안성에 문제가 생긴다. 불필요한 객체 생성은 그저 코드 형태와 성능에만 영향을 미칠뿐이다.**



### 참고
[Defensive copying](http://www.javapractices.com/topic/TopicAction.do?Id=15)


