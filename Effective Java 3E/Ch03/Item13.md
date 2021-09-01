# 아이템 13 : clone 재정의는 주의해서 진행하라
`Cloneable`은 복제해도 되는 클래스임을 명시하는 인터페이스이다.

하지만 `clone`메서드가 선언된 곳은 `Cloneable`이 아닌 `Object`이고 `protected`이다. 따라서 `Cloneable`을 구현하는 것 만으로는 외부 객체에서 `clone`메서드를 호출할 수 없다.

## Cloneable 인터페이스
이 인터페이스는 `Object`의 `protected`메서드인 `clone`의 동작 방식을 결정한다.

`Cloneable`을 구현한 클래스의 인스턴스에서 `clone`을 호출하면 해당 객체의 필드들을 하나씩 복사한 객체를 반환하고, 구현하지 않은 객체에서 호출하면 `CloneNotSupportedException`을 던진다.

**`Cloneable`을 구현할 클래스는 `clone`메서드를 `pulbic`으로 제공하며 사용자는 복제가 제대로 이뤄지리라 기대한다**. 하지만 이를 만족하려면 생성자를 호출하지 않고도 객체를 생성할 수 있게 되는 굉장히 모순적인 메커니즘이 발생한다.

## Object 명세에서의 clone()
이 객체의 복사본을 생성해 반환한다. `복사`의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적은 의도는 다음과 같다.
```java
x.clone() != x // true!!
```
아래도 참이지만 반드시 만족해야하는 것은 아니다.
```java
x.clone().getClass() == x.getClass()
```
다음 식도 일반적으로 참이지만 필수는 아니다.
```java
x.clone().equals(x)
```
관례상, 이 메서드가 반환하는 객체는 `super.clone()`을 호출해 얻어야한다. 이 클래스와 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.
```java
x.clone().getClass() == x.getClass()
```
관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 `super.clone()`으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

하지만 만약, 클래스 A를 상속받는 클래스 B가 있다고 가정해보자. 클래스 B에서 `super.clone()`을 호출한다면 사용자의 의도는 클래스 B 인스턴스를 복사하는 것이지만 실제로 반환되는 것은
클래스 A의 인스턴스가 반환되는 문제가 발생할 것이다.

클래스 A가 `final`클래스라면 문제가 되지 않는다.

정상 동작하는 `clone`을 재정의한 클래스를 만들어보자.
```java
public final class PhoneNumber implements Cloneable {
    .....
    .....
    @Override public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();  // 일어날 수 없는 일이다.
        }
    }
}
```
`Object`의 `clone()`은 `Object`를 반환하기에 반환전에 `PhoneNumber`로 형변환해주었다. 이는 `java`가 공변 반환 타이핑을 지원하므로 가능하다.
* 공변성 : 자기 자신과 자식만 허용 `<? extends T>`
* 반공변성 : 자기 자신과 부모만 허용 `<? super T>`
* 무공변성 : 자기 자신만 허용

## 가변 객체 참조
```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty() {
        return size ==0;
    }

    // 원소를 위한 공간을 적어도 하나 이상 확보한다.
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
위와 같은 `Stack` 클래스를 예시로 든다. 만약 해당 클래스의 `clone`이 단순히 `super.clone()`을 그대로 반환한다면 문제가 발생한다.

`elements`필드는 원본 `Stack`인스턴스와 똑같은 배열을 참조하기 때문에 원본/복제본 중 하나를 수정하면 다른 하나에도 `side effect`를 끼친다.

따라서, `clone` 메서드가 사실상 생성자와 같은 효과를 내도록 해야한다. `clone`은 원본 객체에 아무 영향도 끼치지 않아야하며 복제된 객체의 불변식을 보장해야한다.
이를 만족하려면 `Stack`의 `clone`은 스택 내부 정보를 복사해야하는데, 가장 쉬운 방법은 `elements`배열의 `clone을 재귀적으로 호출해주는 것이다.
```java
    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```
만약 `elements`가 `final`이라면 이 방식은 동작하지 않는다. 따라서 `Cloneable`을 구현한다면 `final`을 제거해야할 수도 있다.

### deepCopy
`clone`을 재귀적으로 호출하는 것만으로 충분하지 않을 때도 있다. 

배열 안에 가변 참조 객체가 있는 구조를 예시로 들고있다.
```java
public class HashTable implements Cloneable {
  private Entry[] buckets = ...;
  private static class Entry {
    final Object key;
    Object value;
    Entry next;
    // contstructor...
  }
  
  @Override public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = buckets.clone();
      return result;
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
}
```
여기서 `clone`을 통한 복제본은 자신만의 버킷 배열을 갖지만 원본과 같은 연결 리스트를 참조하기 때문에 원하지 않은 방향으로 동작할 수 있다.

따라서, 각 버킷을 구성하는 연결 리스트또한 복사해야한다.
```java
Entry deepCopy() {
    return new Entry(key, value, next == null ? null : next.deepCopy());
  }
  @Override public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = new Entry[buckets.length];
      for (int i = 0; i < buckets.length; i++) 
        if (buckets[i] != null)
          result.buckets[i] = buckets[i].deepCopy();
      return result;
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
```
비어있지 않은 버킷에 대해 깊은 복사를 수행하고 이는 자신이 가르키는 연결리스트를 복사하기 위해 자신을 재귀적으로 호출한다.

하지만 연결 리스트를 복제하는 방법은 재귀 호출로 인해 리스트의 원소 수만큼 호출되기 때문에 리스트가 길면 `StackOverFlow`가 발생할 수 있기 때문이다.

이를 해결하기 위해서는 재귀 호출대신 반복자를 사용해 순회하는 방법을 사용하면 된다.
```java
Entry deepCopy() {
  Entry result = new Entry(key, value, next);
  for (Entry p = result; p.next != null; p = p.next)
    p.next = new Entry(p.next.key, p.next.value, p.next.next);
  return result;
}
```
### 고수준 API 활용
`HashTable`에서 `put`을 사용해 복제를 할 수도 있지만 이는 느리다.

`put` 메서드 내부적으로 연관된 엔트리들을 연결하는 과정이 추가적으로 들어가야하기 때문에

### `clone`에서는 재정의 될 수 있는 메서드를 호출하면 안된다.
하위 클래스가 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 복제본의 상태가 달라질 수 있다.

### `clone`을 재정의 할 때는 public, throw X

### 하위 클래스에서 `Cloneable`을 지원하지 못하게 막기
`clone`을 동작하지 않게 구현하여 하위 클래스에서의 재정의를 막을 수 있다.
```java
@Override
protected final Object clone() throws CloneNotSupportedException {
  throw new CloneNotSupportedException();
}
```
### `clone`의 동기화
`Object` `clone`은 동기화를 신경쓰지 않았기 때문에 `Cloneable`을 구현한 스레드 안전 클래스를 작성할 때는 `clone`역시 적절히 동기화가 필요하다.

## 정리
**`Cloneable`을 구현하는 모든 클래스는 `clone`을 제정의해야한다.**

## 복사 생성자와 복사 팩토리를 사용하자!
**복사하는 대상이 배열이 아니라면 복사 생성자와 복사 팩토리를 사용하는 것이 좋다.**

복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 뜻한다.
```java
public class Teacher{
    String name;
    int age;
    List<Student> studentList;
}

public class Student{
    String name;
    int age;
}

// -----------------------------
public class Teacher{
    String name;
    int age;
    List<Student> studentList;

    public Teacher(){
    }
 
    public Teacher(Teacher src){ // 복사 생성자
        this.name = new String(src.name);
        this.age = src.age;
        this.studentList = new ArrayList<>();
        for(int i = 0; i < src.studentList.size(); i++){
            this.studentList.add(new Student(src.studentList.get(i));
        }
    }
}

public class Student{
    String name;
    int age;

    public Student(){
    }

    public Student(Student src){ // 복사 생성자
        this.name = new String(src.name);
        this.age = src.age;
    }
}
```
복사 샏성자를 변형한 복사 팩토리이다. `Collections`에서도 이미 이런 방식으로 구현되어 있다. 아래는 `Collections`의 `copy` static 메서드이다.
```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        int srcSize = src.size();
        if (srcSize > dest.size())
            throw new IndexOutOfBoundsException("Source does not fit in dest");

        if (srcSize < COPY_THRESHOLD ||
            (src instanceof RandomAccess && dest instanceof RandomAccess)) {
            for (int i=0; i<srcSize; i++)
                dest.set(i, src.get(i));
        } else {
            ListIterator<? super T> di=dest.listIterator();
            ListIterator<? extends T> si=src.listIterator();
            for (int i=0; i<srcSize; i++) {
                di.next();
                di.set(si.next());
            }
        }
    }
```

복사 생성자와 복사 팩터리는 `Cloneable/clone`방식보다 나은 점이 많다.
* 생성자를 쓰지 않는 모순적 메커니즘을 벗어난다.
* 엉성한 규약에 기대지 않는다.
* 정상적인 `final`필드 용법과 충돌하지 않는다.
* 불필요한 검사 예외를 던지지 않는다.
* 형변환이 필요하지 않다.
* 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다.
  - `HashSet` 객체를 `TreeSet`타입으로 복제할 수 있다.



