# 아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

## 태그 달린 클래스
두가지 이상의 의미를 표현하기 위해 현재 표현하는 것을 태그로 알려주는 클래스
```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE }

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

현재 태그 값이 `RECTANGLE`, `CIRCLE` 2개뿐이지만 코드가 위와 같이 매우 복잡해지고 여러 문제들이 존재한다.
- `enum` 타입 선언
- 태그 필드
- `switch`문
- 쓸데없는 메모리 소모
- 만약 필드를 `final`로 선언하려면 쓰지 않는 필드까지 생성자에서 초기화해야함
- 컴파일 에러보다는 대부분 런타임 에러가 발생
- 인스턴스 타입만으로는 나타내는 의미를 알 수 없음

만약에 태그 값이 10개, 100개, 1000개라면?🤬🤬

#### 태그 달린 클래스는 장황하고 오류를 내기 쉽고 비효율적!!

<br>

## 클래스 계층구조
가장 상위 `root`가 될 `abstract class`를 정의한다.

내부에는 모든 하위 클래스에서 공통으로 사용될 메소드를 `abstract method`로 추가한다.

하위 클래스는 `abstract class`를 상속하여 계층 구조로 정의된다.

각 하위 클래스는 자신만의 필드를 선언할 수 있어 태그 달린 클래스처럼 불필요한 초기화, 쓸데없는 메모리 소모가 사라진다.

```java
// 상위 root 클래스
abstract class Figure {
    abstract double area();
}

// Circle 클래스
class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}

// Rectangle 클래스
class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}
```

#### 간결하고 명확하며 쓸데없는 코드를 제거한다.

각 필드를 모두 `final`를 사용했기 때문에 생성자에서 초기화를 하였는지 컴파일 레벨에서 체크해준다.

계층구조라서 `root`를 상속하는 하위 클래스를 상속하는 최하위 클래스를 만들 수도 있다.

```java
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```


