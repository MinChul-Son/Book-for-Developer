# 아이템 16 : public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

패키지 외부에서 접근할 수 있는 `public`클래스라면 접근자를 제공하여 클래스 내부 표현 방식을 바꿀 수 있는 유연성을 가지게하는 것이 좋다.

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

`public` 클래스의 필드를 `public`으로 설정하면 이를 사용하는 클라이언트가 생길 것이고 내부 표현 방식을 마음대로 바꿀 수 없게 된다.

하지만, `package-private` 클래스 or `private` 중첩 클래스라면 데이터 필드를 노출한다 해도 문제가 없다.

클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하지만 클라이언트도 이 클래스를 포함하는 패키지 안에서만 동작하는 코드이기 때문에 바깥 코드는 전혀 손대지 않고 내부 표현 방식을 변경할 수 있다.

`java.awt.package` 패키지의 `Point`와 `Dimension` 클래스는 `public` 클래스의 필드를 노출하고 있어 심각한 성능 문제를 유발하고 있다.

`public` 클래스의 필드를 `final` 불변으로 설정한다해도 직접 노출할 때의 단점이 조금은 줄어들지만 여전히 `API`를 변경하지 않고는 표현 방식을 바꿀 수 없고 필드를 읽을 때 부수 작업을
수행할 수 없다. 하지만 불변식은 보장할 수 있다.

아래 `Time` 클래스는 각 인스턴스가 유효한 시간을 표현하는 것을 보장한다.

```java
public final class Time {
    private static final int HOURS_PER_DAY    = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("Hour: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("Min: " + minute);
        this.hour = hour;
        this.minute = minute;
    }

    // 나머지 코드 생략
}
```
