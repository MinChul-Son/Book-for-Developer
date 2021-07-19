# 아이템 1 : 생성자 대신 정적 팩터리 메서드를 고려하라
일반적으로 클라이언트가 클래스의 인스턴스를 얻는 방법은 `public 생성자`이다.

다른 방법으로는 클래스에 `정적 팩터리 메서드(static factory method)`를 제공할 수 있다.
* 클래스의 인스턴스를 반환하는 단순한 정적 메서드

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
} 
```
* 이 정적 메서드는 새로운 객체 인스턴스를 생성해 반환하는 것이 아니라 단순히 Boolean의 상수 객체를 반환한다.

#### 여기에는 장단점이 존재하는데 다음과 같다.

----------------------------------------------
----------------------------------------------
## 장점 1 : 이름을 가질 수 있다.
생성자에는 이름을 지정해줄 수 없다. 따라서 생성자에 넘기는 매개변수와 생성자 자체만으로 반환될 객체의 특성을 제대로 설명하지 못한다.

하지만 정적 팩터리는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.
```java
BigInteger(int bitLength, int certainty, Random rnd); // 생성자 : Constructs a randomly generated positive BigInteger that is probably prime, with the specified bitLength.

BigInteger.probablePrime(int bitLength, Random rnd); // static 메서드 : Returns a positive BigInteger that is probably prime, with the specified bitLength.
```
* 여기서 어느 쪽이 `값이 소수인 BigInteger를 반환한다`는 의미를 잘 설명할 것 같은가?

생성자에는 또다른 제약이 존재하는데, **하나의 시그니처로는 생성자를 하나만 만들 수 있다.**
```java
public class Item1 {
    String name;
    String address;

    public Item1(String name) {
        this.name = name;
    }

    public Item1(String address) { // 하나의 시그니처로는 생성자를 하나만 만들 수 있음 ==> 컴파일 에러 발생
        this.address = address;
    }
}
```
정적 팩터리 메서느는 이런 제약이 없기 때문에 시그니처가 같은 생성자가 여러개 필요할 것 같으면, 생성자를 정적 펙터리 메서드로 바꾸고 각각의 차이를 잘 드러내는 이름을 지어주자.


## 장점 2 : 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
`불변 클래스(immutable class)`는 인스턴스를 미리 만들어 놓고나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 막을 수 있다.

위에서 언급한 Boolean.valueOf(boolean)은 객체를 생성하는 것이 아니라 상수 객체를 반환해준다. (생성 비용이 큰) 같은 객체가 자주 요청되는 상황이라면 정적 팩터리 메서드는 성능을 매우
끌어올려 준다. (Flyweight pattern도 이와 비슷한 기법)
* 싱글톤도 이와 비슷한 기법이라고 생각함(내 의견)

반복되는 요청에 같은 객체를 반환하여 **언제 어느 인스턴스를 살아 있게 할지를 철저히 통제**할 수 있다. 이를 `인스턴스 통제(instance-controlled) 클래스`라고 한다.
인스턴스를 통제하면 `싱글톤(singleton)`으로 만들 수도, `인스턴스화 불가(noninstantiable)`로 만들 수도 있다.

또한 동치인 인스턴스가 단 하나뿐임을 보장할 수도 있다.(a==b일 때만 a.equals(b)가 성립)

참고로 **열거 타입(Enum)은** 인스턴스가 하나만 만들어짐을 보장한다.

## 장점 3 : 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
이 능력은 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 **엄청난 유연성**을 선물한다.

즉, 리턴 타입의 하위 타입 인스턴스를 만들어 반환해줘도 된다는 의미. 리턴 타입은 인터페이스로 지정하고 그 인터페이스의 구현체는 API로 노출시키지 않고도 그 객체를 반환할 수 있어
API를 작게 유지할 수 있다. 이는 인터페이스를 정적 팩터리 메서드의 반환 타입으로 사용하는 **인터페이스 기반 프레임워크**의 핵심 기술이다.

자바8 이후부터는 인터페이스에 정적 메서드를 선언할 수 있다.(자바 9에서는 private 정적 메서드까지도 허용) 이를 활용한 예시가 바로 `java.util.Collections`이다.

`java.util.Collections`는 45개나 되는 인터페이스의 구현체 인스턴스를 제공하지만 그 구현체는 전부 **non-public**이다. 인터페이스 뒤에 숨겨져 있으므로 public으로 제공해야할
API를 줄이고 개념적인 무게(conceptual weight)까지 줄였다. 사용자는 해당 구현체로 어떤 것이 반환되었는지를 신경쓸 필요없이 사용하면 된다.(**매우 유연함**)
또한 클라이언트는 언은 객체를 그 구현체가 아닌 인터페이스만으로 다루게 되는 것이다.
![image](https://user-images.githubusercontent.com/60773356/126115422-614e48de-aeb4-4ca7-9d36-59d4f972a4ef.png)
* 실제 Collections 클래스의 unmodifiableList라는 정적 팩토리 메서드이다.
* 반환 타입이 List이지만 List의 하위 객체를 반환하여도 괜찮다.


## 장점 4 : 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
위에서 언급한 것처럼 반환되는 객체의 타입은 다를 수 있다. `EnumSet`클래스는 생성자 없이 public static 메서드, `allOf()`, `of()`등을 제공한다.
Enum타입의 원소가 64개 이하이면 원소들을 long 변수 하나로 관리하는 `RegularEnumSet`의 인스턴스를, 65개 이상이면 long배열로 관리하는 `JumboEnumSet`의 인스턴스를 반환한다.

클라이언트는 이 두 클래스의 존재를 모르기 때문에, 추후에 변화에 따라 새로운 클래스를 추가하거나 기존의 클래스를 삭제하여도 문제가 되지 않는다.
클라이언트는 반환된 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없다. 단지 `EnumSet`의 하위 클래스이기만 하면 된다.
 
## 장점 5 : 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.    
이는 **서비스 제공자 프레임워크**를 만드는 근간이 된다. 책에서는 `JDBC`를 예시로 들었다.
서비스 제공자 프레임워크는 3개의 핵심 컴포넌트로 구성
1. 서비스 인터페이스 : 구현체의 동작을 정의
2. 제공자 등록 API : 제공자가 구현체를 등록할 때 사용
3. 서비스 접근 API : 클라이언트가 서비스의 인스턴를 얻을 때 사용

여기서 서비스 접근 API가 **유연한 정적 팩터리**이다. JDBC를 사용할 때 getConnection을 통해 Connection 객체을 가져오는데 이 때 사용자가 `mysql`, `h2`, `oracle`등등
어떤 DB를 사용하는 지에 따라서 이 Connection 객체는 달라진다. 이것이 바로 핵심이자 유연함이다. 구현체의 조건을 명시하지 않으면 기본 구현체를 반환한다.
```java
// 출처 : 자바봄 스터디
public class JdbcSample {

    public void jdbcExample() throws SQLException {
        /**
         *  implements java.sql.Driver
         *  java.sql.Driver: 서비스 제공자 인터페이스
         *  com.mysql.cj.jdbc.Driver 는 이 인터페이스를 구현하여 제공자 등록 API 를 통해 등록되고 갈아끼워진다.
         */
        Driver driver = new Driver();
        DriverManager.registerDriver(driver); // 제공자 등록 API

        /** Class.forName("com.mysql.cj.jdbc.Driver") 을 통해 Driver 등록도 가능하다.
         *  DriverManager 는 static 필드에서 Driver 를 등록받기 때문에
         *  ClassLoader 수행 시 mysql Driver 를 주입받는다. 본 예제에서는 registerDriver API 를 사용한다.
         */

        Connection connection = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/jdbc?serverTimezone=UTC", "root", "password"); // 서비스 인터페이스

        Statement statement = connection.createStatement();
        ResultSet resultSet = statement.executeQuery("SELECT * FROM test");


        while(resultSet.next()){
            System.out.println(resultSet.getString(1));
            System.out.println(resultSet.getString(2));
        }


    }
}
```

부가적으로, 서비스 인터페이스의 인스턴스를 제공하는 `서비스 제공자 인터페이스`를 만들 수도 있는데, 그게 없는 경우에는 `리플랙션`을 사용해서 구현체를 만들어 준다.
자바 6부터는 `java.util.ServiceLoader`라는 일반적인 용도의 서비스 프로바이더를 제공하지만, `JDBC`가 그 보다 이전에 만들어졌기 때문에 `JDBC`는 `ServiceLoader`를 사용하진 않는다.

 
-----------------------
## 단점 1 : 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.(=상속할 수 없다)
`Collections 프레임워크`에서 제공하는 유틸리티 구현체(`java.util.Collections`)는 상속할 수 없다.

하지만 이 제약은 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야하므로 오히려 장점이라고 볼 수도 있다.

## 단점 2 : 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
생성자처럼 API설명에 명확히 드러나지 않으므로, 클래스나 인터페이스 문서 상단에 static 팩토리 메서드에 대한 문서를 제공하는 것이 좋다.
![image](https://user-images.githubusercontent.com/60773356/126118365-de043f52-039f-4281-87af-d1d533753854.png)
* `SpringApplication` 클래스에는 정적 메서드 `run()`이 존재한다. 사진을 보면 `run()`에 대한 설명을 주석으로 작성해 놓은 것을 확인할 수 있다.

