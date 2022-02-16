# 아이템 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

`Enum` 타입은 해당 상수가 몇 번째 위치인지를 반환하는 `ordinal`이라는 메서드를 제공한다.

만약 이를 정수값과 연결하려할 때 사용할 수도 있을 것이다.

```java
public enum Number {
    ONE, TWO, THREE, FOUR, FIVE, SIX;
    
    public int number() {
        return ordinal() + 1;
    }
}
```
~~이렇게 쓰는 사람이 있을까?~~

하지만 여기에는 몇가지 문제점이 존재한다.
- 상수 선언 순서를 바꾸는 순간 오작동할 수 있다.
- 이미 사용 중인 정수와 값이 같은 상수는 추가할 수 없다.
- 값을 중간에 비워둘 수도 없다.

**이런 경우에는 `ordinal`을 사용하지말고 인스턴스 필드에 저장하자.**

```java
public enum Number {
    ONE(1), TWO(2), THREE(3), FOUR(4), FIVE(5), SIX(6);
    
    private final int number;
    
    Number(int number) {
        this.number = number;
    }
    
    public int getNumber() {
        return number;
    }
}
```
