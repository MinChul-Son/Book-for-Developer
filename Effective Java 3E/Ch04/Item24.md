# 아이템 24. 멤버 클래스는 되도록 static으로 만들라
`중첩 클래스`는 다른 클래스 안에 정의된 클래스를 말한다.

<br>

## 중첩 클래스
- 정적 멤버 클래스
- 비정적 멤버 클래스
- 익명 클래스
- 지역 클래스

여기서 `정적 멤버 클래스`를 제외하고는 모두 `내부 클래스`에 해당한다.

<br>

### 정적 멤버 클래스
정적 멤버 클래스는 다른 클래스 내부에 선언된다.

바깥 클래스 멤버 중 `static`인 멤버에만 접근할 수 있고, 바깥 클래스의 인스턴스가 생성되지 않아도 정적 멤버 클래스를 생성할 수 있다.
```java
public class Outer {
    private static String foo;

    private static class Inner {
        private String hello = foo;
    }

    public static void main(String[] args) {
        Inner inner = new Inner(); // Outer 생성 없이 바로 정적 멤버 클래스 생성가능
    }
}

```
만약 `Outer`의 멤버 `foo`가 `static`이 아니라면 컴파일 에러가 발생한다.

정적 멤버 클래스는 바깥 클래스와 함께 쓰일 때만 유용한 `public` 도우미 클래스로 자주 쓰인다.

<br>

### 비정적 멤버 클래스
비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다.

따라서 정규화된 `this`를 사용하여 바깥 인스턴스의 메소드나 필드를 참조할 수 있다.
```java
public class Outer {
    private static String foo;
    private String foo2;

    private class Inner {
        private String hello = foo;
        private String hi = foo2;
        private String spring = Outer.this.foo2; // 정규화된 this
    }

    public static void main(String[] args) {
        Outer outer = new Outer();
        Outer.Inner inner = outer.new Inner();
    }
}
```
정규화된 `this`를 사용할 수 있고, 바깥 클래스의 모든 멤버에 접근할 수 있다.

하지만 외부 클래스가 생성되어야만 내부 클래스를 생성할 수 있다.

이는 `GoF Design Pattern`중 `Adaptor` 패턴을 정의할 때 자주 사용된다.

어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 용도이다.

예시로써, `Map`, `Set`, `List` 인터페이스의 구현체들이 내부에서 비정적 멤버 클래스를 주로 사용한다.

<br>

--------------------------------

#### 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 정적 멤버 클래스로 만들자!!

`static`을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 되므로 시간과 공간이 소비된다.

바깥 인스턴스를 참조하는 내부 인스턴스가 생기기 때문에 `GC`의 대상에서 제외되므로 메모리 누수가 생길 수도 있다.

`private` 정적 멤버 클래스는 바깥 클래스가 표현하는 객체의 구성요소를 나타낼 때 주로 사용한다.

`Map` 구현체들은 `key-value`를 표현하는 `Entry`객체를 내부에 가지고 있다.

`Map`과 연관되어 있지만 `Entry`는 `Map`을 직접 사용하지 않기 때문에 `static`을 붙히는 것이 적합하다.

만약, 멤버 클래스가 공개 API의 `public`이나 `protected`라면 `static` 여부는 매우 중요해진다.

멤버 클래스 또한 공개 API가 되고 이를 외부에서 사용하게 된다.

만약 추후에 멤버 클래스에 `static`을 붙혀 정적 멤버 클래스로 바꾼다면? **하위 호환성이 깨질 것이다.**
- 비정적과 정적일 때 객체 생성 방법이 달라 모두 컴파일 에러가 발생할 것임

<br>

### 익명 클래스
익명 클래스는 바깥 클래스의 멤버가 아니고, 쓰이는 시점에 선언과 인스턴스가 생성된다.

또한 이는 코드의 어디서든 만들 수 있으며 비정적 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다.

하지만 상수외에는 정적 멤버를 가질 수 없다.
```java
interface Operator {
    int plus();
}

// 비정적 문맥에서는 바깥 클래스의 인스턴스를 참조할 수 있다!
public class AnonymousExample {
    int x;
    int y;

    public AnonymousExample(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    public int nonStaticMethod() {
        Operator operator = new Operator() {
            private static final String foo = "foo";

            @Override
            public int plus() {
                return x + y;
            }
        };

        return operator.plus();
    }
}

// 정적 문맥에서는 바깥 클래스의 인스턴스를 참조할 수 없고, 상수 외의 정적 필드를 가질 수 없다.
public class AnonymousExample {
    int x;
    int y;

    public AnonymousExample(int x, int y) {
        this.x = x;
        this.y = y;
    }

    //정적인 구문
    public static void staticMethod(){
        Operator operator = new Operator() {
            private static final String foo = "foo";
            private static String cant = "cant";  // 컴파일 에러! 이너 클래스는 정적 변수를 가지고 있을수 없다!
            @Override
            public double plus() {
                return x + y; // 컴파일 에러! 바깥 클래스의 인스턴스를 참조할 수 없다.
            }
        };
    }
}
```
이외에도 많은 제약이 따른다.
- 선언한 지점에서만 인스턴스 생성 가능
- `instanceOf`나 클래스의 이름이 필요한 작업 수행 불가
- 여러 인터페이스를 구현할 수 없음
- 인스턴스를 구현하면서 클래스를 상속할 수도 없음
- 상위 타입에서 상속한 멤버 외에는 호출할 수 없음
- 가독성이 떨어짐

`Lambda`의 등장으로 잘 사용되지 않는다.

<br>

### 지역 클래스
지역 변수를 선언할 수 있는 곳이라면 어디서든 선언할 수 있으며 `Scope`도 지역 변수와 같다.

멤버 클래스처럼 이름이 존재하며 반복해서 사용할 수 있다.

비정적 문맥에서 사용될 때만 바깥 클래스를 참조할 수 있다.

정적 멤버는 가질 수 없다.

