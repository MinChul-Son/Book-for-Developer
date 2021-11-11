# 아이템 25. 톱레벨 클래스는 한 파일에 하나만 담으라
파일 하나에 톱레벨 클래스를 여러 개 선언하면 한 클래스를 여러가지로 정의할 수 있다.(같은 클래스가 여러 곳에 존재할 수 있다는 의미)
- 어떤 것이 먼저 컴파일되느냐에 따라 다름

```java
// Utensil.java 파일 내부
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```

```java
// Dessert.java 파일 내부
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

<br>

톱레벨 클래스들을 각자 분리시켜면 문제는 해결된다.

#### 여러 톱레벨 클래스를 하나에 담고 싶다면 정적 멤버 클래스를 고민해보자!

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```
`private`를 사용해 접근 제어까지 깔끔하게 처리할 수 있다.