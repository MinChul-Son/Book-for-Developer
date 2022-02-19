# 아이템 36. 비트 필드 대신 EnumSet을 사용하라

**구닥다리 비트 필드 열거 상수**
```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0;
    public static final int STYLE_ITALIC = 1 << 1;
    public static final int STYLE_UNDERLINE = 1 << 2;
    public static final int STYLE_STRIKETHROUGH = 1 << 3;
}
```

비트 필드는 앞서서 알아봤던 정수 열거 상수의 단점을 그대로 가진다.

추가로, 피드 필드 값이 그대로 출력될 때 해석하기 더 어렵고, 모든 원소를 순회하기도 까다롭다는 단점이 있다.

또한, 최대 몇 비트가 필요한지를 미리 예측하고 적절한 타입을 선택해야한다.

<br>

## EnumSet
![스크린샷 2022-02-19 오후 2 32 10](https://user-images.githubusercontent.com/60773356/154787796-bb825f5d-36aa-46d7-83a8-1e1945f60bab.png)

`EnumSet`은 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.

당연하게도 `Set` 인터페이스를 구현했기 때문에 어떤 `Set` 구현체와도 함께 사용할 수 있다. 

`EnumSet`의 내부는 비트 벡터로 구현되었기 때문에 원소가 총 64개 이하라면 비트 필드에 비견되는 성능을 보여줄 수 있다.

```java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }

    // 사용 예
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```

아직 `Enumset`을 불변으로 만드는 API는 존재하지 않는다.

하지만 `Collections.unmodifiableSet`을 사용하며 방어적 복사로 `EnumSet`을 감싸 불변으로 만들 수는 있다!
