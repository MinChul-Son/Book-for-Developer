# 아이템 53. 가변인수는 신중히 사용하라

가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다.

이는 인수의 개수만큼 해당 타입의 배열을 만들고 인수를 저장하여 메서드에 건내주는 방식으로 동작한다.

```
static int sum(int... args) {
    int sum = 0;
    
    for (int arg : args)
        sum += arg;
    
    return sum;
}
```

만약 해당 메서드가 한개 이상의 인수를 필수로 받아야한다면 어떨까?

아래와 같은 상황이다.

```java
public class Varargs {
    static int min(int... args) {
        if (args.length == 0)
            throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
        int min = args[0];
        for (int i = 1; i < args.length; i++)
            if (args[i] < min)
                min = args[i];
        return min;
    }
}
```
인자가 하나 이상 넘어오는지를 배열의 길이를 확인하는 것으로 검증한다.

이 때 인수의 개수가 몇개인지는 런타임 시점에 알 수 있기 때문에 컴파일 시점에는 오류를 찾을 수 없다.


## 해결책
매개변수를 2개 받도록 하면 된다.

```java
public class Varargs {
    static int min(int firstArg, int... remainingArgs) {
        int min = firstArg;
        for (int arg : remainingArgs)
            if (arg < min)
                min = arg;
        return min;
    }

    public static void main(String[] args) {
        System.out.println(sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));
        System.out.println(min(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));
    }
}
```
첫번째 인수를 입력하지 않으면 컴파일 에러가 나기 때문에 더 빠른 시점에 동작을 확인할 수 있다.

## 가변인수로 인한 성능 저하
가변인수는 호출될 때마다 배열을 새로 생성하기 때문에 성능에 민감한 상황이라면 성능저하가 발생할 수 있다.

이 때는 다중정의를 통해 해결하자.

`EnumSet`의 정적 팩토리 메서드도 이 기법을 사용해 생성 비용을 최소화하고 있다.

<img width="668" alt="스크린샷 2022-03-27 오후 3 43 28" src="https://user-images.githubusercontent.com/60773356/160270386-031a86dd-50b6-4aa2-a238-0a6a6e0c1ed1.png">

