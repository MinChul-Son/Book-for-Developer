# 아이템 18. 상속보다는 컴포지션을 사용하라
**상속**은 코드를 재사용하는 강력한 수단이지만 항상 최선은 아니다.

상위 클래스가 애초에 상속을 위해 설계되었고 문서화도 잘되어있다면 안전하지만 그렇지 않다면 위험하다.

<br>

## 상속의 위험성
### 메소드 호출과 달리 상속은 캡슐화를 깨뜨린다.
**상위 클래스의 구현에 따라 하위 클래스 동작에 이상이 생길 수 있다.**

원래는 아무 문제없이 동작하는 하위 클래스가 상위 클래스의 새로운 릴리즈로 인해 오작동할 수 있다는 의미이다.

책에서 말하고 있는 예시를 살펴보자.

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```
위 코드에서 `s.getAddCount`의 출력결과는 3을 예상하지만 실제로는 6을 반환한다. 

![image](https://user-images.githubusercontent.com/60773356/139582884-a752fdd5-73b8-40db-b422-0bfe171c3818.png)

`addAll` 메서드는 내부적으로 `add` 메서드를 사용하는데 위의 코드에서 `add`를 `@Override`했기 때문에 문제가 발생한다.

`addAll`을 호출하여 3을 더하지만 `add`가 호출되기 때문에 3이 한번 더 더해져 결과적으로 6을 반환한다.

즉, `addAll`로 추가한 원소당 2씩 증가하는 것이다.

<br>

#### `addAll`을 재정의하지 않으면 되지 않나? 

위와 같이 생각할 수는 있지만 이는 `addAll`이 `add`를 사용해 구현되었다는 가정을 하고 있다는 한계가 있다.

즉, 추후에 변경된다면? 다시 문제가 발생할 가능성이 존재한다.

<br>

#### 그럼 다른 방법으로 재정의하면?

이 방법이 차라리 위보다는 나은 방법이지만 상위 클래스의 메소드를 직접 구현하는 방식은 어렵고, 오류나 성능 저하가 발생할 수 있다.
- 전세계적인 자바 고수들이 모여 만든 클래스와 메소드들인데 일반 사용자들이 이를 따라갈 재간이나 있을까? 

<br>

#### 또한, 새로운 릴리스에서 상위 클래스에 새로운 메소드가 추가될 경우에도 하위 클래스가 깨지기 쉽다.

실제로 `Hashtable`과 `Vector`를 컬렉션 프레임워크에 포함시켜 관련 보안 문제를 수정해야하는 사태가 벌어졌다.

만약, 새로운 메소드를 직접 추가하는 방식 또한 새로운 릴리스에서 (그럴 가능성이 희박하지만) 내가 만든 것과 반환 타입만 다르고 동일한 메소드가 추가된다면
직접 만든 것은 컴파일조차 되지 않을 것이다.

<br>

## 컴포지션을 사용하자!!
위의 문제를 모두 피해갈 수 있는 방법은 바로 **컴포지션**을 사용하는 것이다!

기존 클래스를 확장하는 것이 아닌 새로운 클래스를 만들고 `private` 필드로 기존 클래스의 인스턴스를 참조하는 방법을 의미한다.

새 클래스의 인스턴스 메소드는 기존 클래스에 대응하는 메소드를 호출하는 `forwarding`방식을 사용하며 이를 `forwarding method`라고 한다.

먼저 전달 메소드만으로 이루어진 재사용 가능한 클래스 `ForwardingSet`이다.

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```
전달 클래스를 사용하는 집합 클래스이다.

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```

`ForwardingSet`은 `HashSet`의 모든 기능을 정의한 `Set`인터페이스를 활용하여 설계되어있기 때문에 매우 유연하다.

이 방식은 한 번만 구현해두면 어떠한 `Set` 구현체라도 계측할 수 있다.

<br>

다른 `Set` 인스턴스를 감싸고 있다는 뜻에서 `InstrumentedSet`같은 클래스를 **래퍼 클래스`Wrapper Class`**라고 부른다!

또한, 다른 `Set`에 계측 기능을 덧씌운다는 뜻에서 `Decorator Pattern`이라고 한다.

#### 컴포지션 + 전달 = 위임(넓은 의미에서)

`Wrapper Class`의 단점은 콜백 프레임워크와는 어울리지 않는다는 점을 제외하고는 없다.



<br>

실제로 `Stack`은 `Vector`를 확장해선 안됐다.

또한, `Properties`도 `Hashtable`을 확장해서는 안됐다.
- `getProperty(key)`와 `get(key)`의 결과는 다를 수 있다.
    - `getProperty`는 `Properties`의 기본 메소드이고, `get`은 `Hashtable`의 메소드이기 때문
    

### 상속의 주의사항
- 상속은 반드시 하위 클래스가 상위 클래스의 `진짜` 하위 타입인 상황에서만 쓰자.(is-a 관계일 때만)
    - 진짜 하위 클래스라고 확인할 수 없다면 상속을 사용하지 말자.
- 상속이 불필요한 상황에서 사용하는 것은 **내부 구현을 불필요하게 노출**하는 것이다.
    - `API`가 내부 구현에 묶여 성능이 제한된다.
    - 클라이언트가 내부에 직접 접근할 수 있게되는 심각한 문제가 발생한다.
- **상속은 상위 클래스의 결함까지 그대로 물려준다**.
