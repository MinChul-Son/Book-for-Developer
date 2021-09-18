# 아이템 15 : 클래스와 멤버의 접근 권한을 최소화하라
잘 설계된 컴포넌트는 클래스 내부 데이터와 내부 구현 정보를 외부로부터 얼마나 잘 숨겼는지에 따라 판단할 수 있다.(구현과 API를 분리)

오직 `API`를 통해서 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않아야한다.

이것이 바로 `정보 은닉` 혹은 `캡슐화`!!

## 정보 은닉의 장점
* 개발 속도 향상 : 여러 컴포넌트를 병렬로 개발하기 때문
* 관리 비용 감소 : 빠른 디버깅과 교체에 대한 부담이 적음
* 성능 최적화 : 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화 가능
* 재사용성 증대 
* 시스템 제작의 난이도 감소 : 개별 컴포넌트의 동작을 검증할 수 있음

## 접근 제한자
정보 은닉의 핵심은 접근 제한자인 `private`, `package-private`, `protected`, `public`을 활용하는 것이다.
* private : 같은 클래스 내에서만 접근 가능
* package-private : 같은 패키지에서 접근 가능
* protected : 같은 패키지와 상속한 하위 클래스에서도 접근 가능
* public : 모든 곳에서 접근 가능

**모든 클래스와 멤버의 접근성을 가능한 좁혀야한다!** == **올바르게 동작하는 한 가장 낮은 접근 수준을 부여해야한다.**

클래스나 인터페이스에 `public`을 선언하게되면 **공개 API**가 된다. `public`을 선언하면 하위 호환을 위해 항상 관리해줘야한다.

`package-private`(default)로 선언하면 해당 패키지 안에서만 사용할 수 있다. 
따라서, 패키지 외부에서 사용할 일이 없다면 `package-private`으로 선언하자. 이렇게되면 **공개 API**가 아닌 내부 구현이 되므로 언제든 수정이 가능하다.

### private static 중첩 클래스
한 클래스에서만 사용하는 `package-private` 클래스나 인터페이스는 사용하는 클래스 안에 `private static`으로 중첩시키자.

이렇게하면 바깐 클래스 하나에서만 접근 할 수 있다.

```java
public class Outer {
  
  public void method() {
    new Inner().method();
  }
  
  // private static 중첩 클래스
  private static class Inner {
    public void method() {
      System.out.println("Static Nested Class의 인스턴스 메소드 호출");
    }
  }
}
```

### 공개 API
공개 API를 설계한 후에 그 외 모든 멤버는 `private`로 만들자. 그 다음 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한해서만 `package-private`으로 풀어주자.

`private`와 `package-private` 멤버는 모두 내부 구현에 해당하므로 공개 API에 영향을 주지 않는다.

하지만 `Serializable`을 구현한 클래스에서는 의도치 않게 필드들도 공개 API가 될 수 있기에 주의하자.

```java
public class Member implements Serializable {
  private static final long serialID = 1L;
  private String id;
  private String password;
  private String name;
}
```

위와 같은 `Serializable`을 구현한 클래스를 역직렬화하는 과정에서 `private`인 필드가 공개될 수 있다는 의미이다.

`package-private`에서 `protected`로 바꾸면 그 멤버에 접근할 수 있는 범위가 매우 넓어지기 때문에 **`protected` 멤버의 수는 적을 수록 좋다.**

### 리스코프 치환 원칙을 위한 제약
`LSP`를 지키기위해 상위 클래스를 재정의할 때 접근 수준을 상위 클래스보다 더 좁게 설정할 수 없다.(부모 클래스가 `public`일 때 자식 클래스를 `private`으로 하면 컴파일 에러)

## public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.
* 불변을 보장할 수 없다.
*  


