# 아이템 20. 추상 클래스보다는 인터페이스를 우선하라
`Java`가 제공하는 다중 구현 메커니즘은 `interface`와 `abstract class`이다.
- `Java 8`부터는 `interface`에 `default` 메소드 가능

즉, 두 가지 모두 **인스턴스 메소드**를 제공할 수 있다.

하지만, 가장 큰 차이점은 `abstract class`는 `extends` 키워드를 사용하기 때문에 이를 구현하는 클래스는 해당 `abstract class`의 **하위 타입**이 되어야한다.

또한, **단일 상속**만 가능하므로 `abstract class`는 새로운 타입을 정의할 때 큰 제약이 따른다.

<br>

## `interface`의 장점
### 1. 기존 클래스에도 새로운 `interface`를 쉽게 구현해넣을 수 있다.
새로운 `interface`를 정의하고 기존 클래스에 `implements` 키워드를 사용해 해당 `interface`의 메소드를 추가만 하면 된다.

하지만, 새로운 `abstract class`를 기존 클래스가 상속한다면?

기존 클래스는 새로운 `abstract class`의 하위 타입이 되기 때문에 문제가 발생할 수 있다.
- 상속을 사용하기 때문에 계층 구조

<br>

### 2. 믹스인 정의에 적합하다.
**믹스인(mixin)**을 구현한 클래스는 자신의 주된 기능 외에 믹스인 인터페이스에서 제공하는 기능을 할 수 있다.

`interface`는 상속을 사용하지 않기 때문에 계층 구조가 아니므로 유연하게 믹스인을 구현할 수 있다.
- `Comparable`은 자신을 구현한 클래스의 인스턴스끼리 순서를 정할 수 있다고 선언하는 `mixin interface`이다.

<br>

### 3. 계층 구조가 없는 타입 프레임워크를 만들 수 있다.

```java
public interface Singer {
    AudioClip sing(Song s);
}

public interface SongWriter {
    Song compose(int chartPosition);
}

public interface SingerSongWriter extends Singer, SongWriter {
    AudioClip strum();
    void actSensitive();
}
```
`interface`는 다중 상속을 지원하므로 위와 같이 기존 클래스와 동일한 일을 하는 새로운 `interface`로 확장할 수 있다.
- `implements Singer, SongWriter` 또한 가능하다.

이를 `abstract class`를 사용한다면, 각 조합을 지원하는 고도비만 계층구조가 만들어진다. ==> **조합 폭발**
```java
public abstract class Singer {
    AudioClip sing(Song s);
}

public abstract class SongWriter {
    Song compose(int chartPosition);
}

public abstract class SingerPlusSongWriter {
    AudioClip sing(Song s);
    Song compose(int chartPosition);
    AudioClip strum();
    void actSensitive();
}
```
단일 상속만 가능하므로 직접 모든 조합을 `abstract class`로 만든 후 이를 상속하는 별도의 클래스를 통해 사용해야한다.

<br>

### 4. `Wrapper` 클래스와 함께 사용하면 기능을 향상시키는 안전하고 강력한 수단

<br>

### 5. `default` 메소드
구현 방법이 명백하다면 `Java 8`부터 추가된 `default` 메소드를 추가해줄 수 있다.

하지만 이는 동작 과정과 순서, 결과를 문서화하여 제공해주어야한다.(`@implSpec`)

<br>

## `interface`의 제약
### 1. 인스턴스 필드를 가질 수 없다.

<br>

### 2. `public`이 아닌 정적 멤버를 가질 수 없다.(`private` 정적 메소드는 예외)

<br>

### 3. 자신이 만들지 않은 `interface`에는 `default` 메소드를 추가할 수 없다.

<br>

---------------------------------

<br>

### `interface`와 `abstract class`의 장점을 모두 살리는 템플릿 메소드 패턴

```java
public interface Animal {
    void walk();
    void bark();
}

public abstract class AbstractAnimal implements Animal {
    @Override
    public void walk() {
        System.out.println("뚜벅 뚜벅");
    }
    
    public abstract void bark();
}

public class Dog extends AbstractAnimal implements Animal {
    @Override
    public void bark() {
        System.out.println("멍멍!!");
    }
}

public class Cat extends AbstractAnimal implements Animal {
    @Override
    public void bark() {
        System.out.println("야~~옹!");
    }
}

public class AnimalExample {
    public static void main(String[] args) {
        Dog dog = new Dog();
        Cat cat = new Cat();

        dog.walk();
        dog.bark();

        cat.walk();
        cat.bark();
    }
}
```
![image](https://user-images.githubusercontent.com/60773356/140293527-0e459e70-67d0-482b-8cac-e27085c68370.png)

필요하다면 `interface`에서 `default` 메소드를 제공할 수도 있다.

위의 방법을 사용한다면 단순히 `추상 골격 구현`을 확장하는 것 만으로 `interface`를 구현하는데 필요한 대부분의 일이 완료된다.

관례상 `Abstract + 인터페이스 명`으로 골격 구현 클래스를 작명한다.
- `AbstractSet`, `AbstractList` 등등

**골격 구현 클래스는 추상 클래스처럼 구현을 도와주는 동시에 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서 자유롭다!**

구조적으로 골격 구현을 확장하지 못한다면 `interface`를 직접 구현해야한다.

이 경우에도 `default` 메소드의 이점은 여전히 누릴 수 있다.

<br>

### 골격 구현 클래스를 우회적으로 이용 : 시뮬레이트한 다중 상속
`interface`를 구현한 클래스에서 해당 골격 구현을 확장한 `private` 내부 클래스를 정의해 각 메소드 호출을 내부 클래스의 인스턴스에 전달하는 방식이다.

`abstract class`가 이미 다른 클래스를 상속하고 있어 확장이 불가할 때 사용하면 될 것 같다.

```java
public interface Animal {
    void walk();
    void bark();
}

public abstract class AbstractAnimal implements Animal {
    @Override
    public void walk() {
        System.out.println("뚜벅 뚜벅");
    }
    
    public abstract void bark();
}

public class Dog extends Something implements Animal {
    InnerAbstractAnimal innerAbstractAnimal = new InnerAbstractAnimal();

    @Override
    public void walk() {
        innerAbstractAnimal.walk();
    }
    
    @Override
    public void bark() {
        innerAbstractAnimal.bark();
    }

    private class InnerAbstractAnimal extends AbstractAnimal {
        @Override
        public void bark() {
            System.out.println("멍멍!!");
        }
    }
}
```

`Wrapper` 클래스를 사용한 `Forwarding` 방법과 유사하다고 보면된다.


### 골격 구현 작성법
1. `interface`를 살펴 다른 메소드들의 구현에 사용되는 기반 메소드 선정
2. 기반 메소드를 추상 메소드로 작성
3. 직접 구현할 수 있는 메소드는 모두 `default` 메소드로 제공
4. 기반 메소드나 `default` 메소드로 만들지 못한 메소드는 해당 `interface`를 구현하는 골격 구현 클래스를 만들어 모두 넣어준다.

참고로, `interface`의 모든 메소드가 모두 기반 메소드와 디폴트 메소드가 된다면 골격 구현 클래스를 별도로 만들 이유는 없다.

<br>

### 골격 구현 주의 사항
기본적으로 상속하여 사용하는 것을 가정하기 때문에 상속을 고려한 설계와 문서화를 수행해야한다.

<br>

#### 단순 구현은 골격 구현의 작은 변종으로 `AbstractMap.SimpleEntry`가 그 예이다.
- 다른 점은 `abstract class`가 아니라는 점
- 바로 사용할 수 있다.(물론, 확장도 가능)