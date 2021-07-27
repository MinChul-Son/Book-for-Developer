# 아이템 7 : 다 쓴 객체 참조를 해제하라
`java`와 같이 `Garbage Collector`를 가진 언어를 사용한다고 메모리 관리에 신경을 쓰지 않아도 될거라는 생각은 **오산**이다.

책에서 소개한 예제를 살펴보자.
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
```
이 스택을 사용하는 프로그램을 오래 실행하다보면 `GC`와 메모리 사용량이 늘어나 결국 성능이 저하될 것이다.

스택에 여러번 `push`를 하고 다시 여러번 `pop`을 했다고 생각해보자. 아무리 `pop`을 하여도 스택이 차지하고 있는 메모리는 줄지 않는다. 즉, `GC`의 대상이 아니다!
스택의 구현체는 필요 없는 객체(책에서는 이를 다 쓴 참조`obsolete reference`라고 표현)에 대한 레퍼런스를 그대로 가지고 있기 때문이다.

**객체 참조 하나를 살려두면** `Garbage Collector`는 **그 객체뿐 아니라 그 객체가 참조하는 모든 객체(또 그 객체가 참조하는 모든 객체..)를 회수하지 못한다**.

해법은 해당 참조를 다 썼을 때(위 코드에서는 `pop`) `null`처리를 통해 참조를 해제하면 된다.
```java
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object value = elements[--size];
        elements[size] = null;
        return value;
    }
```
`null`로 참조를 해제하기 때문에 다음 `GC`의 대상이 된다. 또한 실수로 `null`처리한 참조를 사용하면 즉시 `NullpointerException`이 발생한다. 이는 잘못된 객체를 돌려주는 것보다 훨씬 좋다.
**프로그래밍 에러는 항상 최대한 빨리 발견하는 것이 좋은 것이다.**

하지만 항상 필요없는 객체를 `null`로 처리하는 것은 바람직하지 않다. **객체 참조를 `null` 처리하는 일은 예외적인 경우여야만 한다.**

다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 특정한 범위(scope)안에서만 사용하는 것이다. local 변수는 해당 scope를 넘어가면 무의미해지기 때문이다.
위의 코드에서는 `size`라는 멤버 변수와 `elements`를 쓰는 경우이므로 예외적인 상황이고, 명시적으로 `null`처리를 해줘야한다.

`Stack`클래스와 같이 **자기 메모리를 직접 관리하는 경우에 `null`처리를 해주어야한다**. `Garbage Collector`는 어떤 객체가 필요없는 객체인지를 알 수 없다.
이런 경우에는 프로그래머가 해당 레퍼런스를 `null`처리하여 `Garbage Collector`에게 필요없는 객체임을 알려주어야 한다.

#### 메모리를 직접 관리하는 클래스는 프로그래머가 메모리 누수를 주의해야한다.



## 캐시
캐시를 사용할 때도 메모리 누수를 주의하여야한다. 객체 참조를 캐시에 넣어두고 캐시를 비우는 것을 잊어버린다면 문제가 발생한다.

만약 캐시 외부에서 `Key`를 참조하는 동안만 `Entry`가 살아 있는 객체가 필요한 상황이라면 `WeakHashMap`을 사용하여 캐시를 만들면 된다.(아래에서 자세히 설명)

또는 시간이 지날수록 `Entry`의 가치를 떨어뜨리는 방식을 사용한다. 이 방식은 쓰지 않는 `Entry`를 주기적으로 비워주어야하는데, `ScheduledThreadPoolExecutor`와 같은 백그라운드 쓰레드를
활용하거나 새로운 `Entry`를 추가할 때 부가 작업으로 기존 캐시를 비우는 방법이 있다.(`LinkedHashMap`은 `removeEldestEntry`메서드를 통해 후자의 방식을 지원함)

## 콜백
리스너 혹은 콜백 또한 메모리 누수의 주범이다. 클라이언트가 콜백을 등록만하고 해지하지 않는다면 콜백이 계속 쌓여갈 것이다.
이또한 `WeakHashMap`을 사용하여 해결할 수 있다.

## Weak References
[참조](https://web.archive.org/web/20061130103858/http://weblogs.java.net/blog/enicholas/archive/2006/05/understanding_w.html)

보통 우리가 객체를 만들때 사용하는 `new`를 사용한 것은 모두 `Strong References`이다. 
```java
Object strong = new Object(); // Strong References
WeakReference weak = new WeakReference(strong);  // Weak References
```
`GC`의 대상이 되기 위해서는 그 객체를 가리키는 참조가 모두 없어져야한다. `Weak References`는 `Strong References`의 참조가 모두 사라지면 `GC`의 대상이 될 수 있다.
따라서 캐시와 콜백에서 `WeakHashMap`을 사용하면 메모리 누수를 막을 수 있다고 얘기하는 것이다.
```java
public class CacheSample {
  public static void main(String[] args) {
    Object key1 = new Object(); // Strong References
    Object value1 = new Object(); // Strong References
    
    Map<Object, Object> cache = new HashMap<>(); // Strong References
    cache.put(key1, value1);
  }
}
```
이 코드에서는 `cache`가 `Strong References`이기 때문에 객체 참조를 `cache`에 넣고 이를 그대로 놔두면 외부에서 `key`의 참조가 사라지더라도 `Strong References`인 `cache`가
`key`를 계속 참조하고 있기 떄문에 `GC`의 대상이 되지 않고 **메모리 누수의 주범**이 된다.
```java
Map<Object, Object> cache = new WeakHashMap<>(); // Weak References
```
이를 `WeakHashMap`을 사용하면 `key`를 참조하는 대상이 없어진다면 `GC` 대상이 되어 `cache`가 자동으로 비워지게 된다.

`자바봄` 스터디의 테스트 예시 코드를 가져왔다.
```java
    @DisplayName("HashMap 테스트")
    @Test
    void test1() {
        //given
        Map<Foo, String> map = new HashMap<>();

        Foo key = new Foo();
        map.put(key, "1");

        //when
        key = null;
        System.gc();

        //then
        assertAll(
                "HashMap 은 참조를 끊어도 map 이 비어있지 않다.",
                () -> assertThat(map.isEmpty()).isFalse()
        );

    }

    @DisplayName("WeakHashMap 테스트")
    @Test
    void test2() {
        //given
        Map<Foo, String> map = new WeakHashMap<>();

        Foo key = new Foo();
        map.put(key, "1");

        //when
        key = null;
        System.gc();

        //then
        assertAll(
                "WeakHashMap 은 참조를 끊으면 map 이 비어있다.",
                () -> assertThat(map.isEmpty()).isTrue()
        );
    }

```
