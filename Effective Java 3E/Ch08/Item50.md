# 아이템 50. 적시에 방어적 복사본을 만들라

클라이언트가 불변식을 깨려는 상황에 대비해 방어적인 프로그래밍을 하자.

주의를 기울이지 않으면 자신도 모르게 내부를 수정하도록 허용할 수 있다.

```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각. 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                    start + "가 " + end + "보다 늦다.");
        this.start = start;
        this.end   = end;
    }

    public Date start() {
        return start;
    }
    public Date end() {
        return end;
    }

    public String toString() {
        return start + " - " + end;
    }
}
```

불행히도 `Java`의 `Date` 클래스는 `setDate`, `setHours`, `setMinutes` 등 불변을 해치는 메서드를 다수 가지고 있다.

불변을 해치는 낡은 `Date` 대신 `LocalDate` 또는 `LocalDateTime`을 사용하자!

<br>

## 방어적 복사
외부의 공격으로부터 내부를 보호하려면 생성자에서 받은 가변 매개변수를 각각 방어적 복사를 하면 된다.

```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각. 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end   = new Date(end.getTime());

        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(
                    this.start + "가 " + this.end + "보다 늦다.");
    }

    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }
    
    public String toString() {
        return start + " - " + end;
    }
}
```

매개변수의 유효성 검사 이전에 방어적 복사를 만들어야 하는데, 멀티스레딩 환경에서 다른 스레드가 원본 객체를 수정할 위험이 있기 떄문이다.

이를 `TOCTOU` 공격이라 부른다.

매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사에 `clone`을 사용해선 안된다.

또한, 접근자에서도 가변 필드는 방어적 복사본을 반환하자.

리스트를 반환할 때, `List.of()`, `unModifiableList()`룰 사용하는 것도 같은 맥락이다.

<br>

항시 객체의 참조를 내부에 저장해야한다면 그 객체가 잠재적으로 변경될 수 있는지를 생각해야한다.

해당 객체가 임의로 변경되었을 때 클래스에 문제가 생길 가능성이 조금이라도 존재한다면 방어적 복사를 수행하자.

<br>

하지만 방어적 복사는 엄밀히 새로운 객체 인스턴스를 생성하는 행위이기에 성능 저하가 따른다. 항상 쓸 수 있는 것도 아니다.

방어적 복사를 사용하지 않을 때는 클라이언트가 해당 가변 객체의 통제권을 완전히 넘겨준다고 생각하더라도 이를 문서화해서 기록해두자.

