# 아이템 9 : try-finally 보다는 try-with-resources를 사용하라
자바 라이브러리에는 `InputStream`, `OutputStream`, `java.sql.Connection`과 같이 `close`메서드를 호출해 직접 닫아줘야하는 자원이 많다.

하지만 이는 클라이언트가 놓치기 쉬워 성능 문제의 원인이 되고 있다. 아이템 8에서 살펴보았듯이 안전망으로 `finalizer`를 사용할 수는 있지만 믿을 수 없다.

## try-finally
```java
static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
```
여기서 자원을 하나 더 사용하게 된다면?
```java
public class Copy {
    private static final int BUFFER_SIZE = 8 * 1024;

    // 코드 9-2 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다! (47쪽)
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
```
코드가 매우 지저분하고 가독성이 떨어진다!!

또한 책에서는 `try-finally`의 문제로 두가지의 예외가 발생했을 경우에 두번째 예외가 첫번째 예외를 집어삼켜버리고 있다고 말한다. 아래 예시 코드를 보자.
```java
MyResource myResource = null;
try {
    myResource = new MyResource();
    myResource.doSomething(); // 예외 발생
} finally {
    if (myResource != null) {
        myResource.close(); // 예외 발생
    }
}
```
만약 위의 코드에서 try블럭의 `doSomething`과 finally블럭의 `close`에서 예외가 발생했다고 가정해보자. 그렇게 된다면, 첫번째 예외인 `doSomething`에서 발생한 예외는
두번째 예외인 `close`에서 발생한 예외로 덮혀 보이지 않게되는데 책에서는 이 점을 이야기하고 있다. 이런 문제가 발생하면 디버깅하기가 매우 어려워진다.
이를 해결하기위해 코드를 수정할 수는 있지만 매우 지저분해진다.


## try-with-resources
[try-with-resources](https://www.baeldung.com/java-try-with-resources)

`자바 7`이 제공하는 `try-with-resources`를 사용하면 이 모든 문제들이 해결된다. 이를 사용하기 위해서는 해당 자원이 `AutoClosable` 인터페이스를 구현해야한다.

우리가 자주 사용하는 자바 라이브러리와 서드파티 라이브러리들의 수많은 클래스와 인터페이스가 이미 `AutoClosable`을 구현하거나 확장해두었다.

```java
// try-finally로 작성한 코드를 try-with-resources로 재작성

static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        }
```

```java
static void copy(String src, String dst) throws IOException {
        try (InputStream   in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
```

`try-with-resources`는 가독성이 높고 문제를 진단하기에도 훨씬 좋다. 만약 `br.readLine()`에서 예외가 발생하고 숨겨져있는 `close`에서도 예외가 발생했다면
`close`에서 발생한 예외는 숨겨지고 `readLine`에서 발생한 예외가 기록된다. 또한 뒤에 발생한 에러는 사라지는 것이 아닌 스택 추적 내역에 `suppressed(숨겨졌다)`는 label을 달고
출력된다. `Throwable`의 `getSuppressed`메서드를 사용해서 뒤에 쌓여있는 에러를 가져올 수도 있다.

`try-finally`처럼 `try-with-resources`도 `catch`절을 쓸 수 있다.
```java
static String firstLineOfFile(String path, String defaultVal) {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            return defaultVal;
        }
    }
```
