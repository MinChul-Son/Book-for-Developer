# 아이템 68. 일반적으로 통용되는 명명 규칙을 따르라

`Java`의 명명 규칙(철자, 문법)을 반드시 따르자!

규칙을 어기면 사용과 유지보수가 어려워진다.

## 철자 규칙

### 패키지, 모듈
패키지와 모듈은 각 요소를 `.`으로 구분해 계층적으로 짓는다.

요소들은 모두 소문자 알파벳(드물게 숫자)로 이뤄진다.

> ex: `java.util`

패키지의 요소는 일반적으로 8자 이하의 짧은 단어로 한다.

여러 단어로 구성된 이름이라면 각 단어의 첫글자만 따서 약어를 사용해도 좋다.

### 클래스, 인터페이스
하나 이상의 단어로 이뤄지며 각 단어는 대문자로 시작한다.(Pascal Case)

> ex: `List`, `Stream`

널리 통용되는 줄임말을 제외하고는 단어를 줄여 쓰지 않도록 한다.


### 메서드, 필드
첫 글자만 소문자로 쓰고 두번째 단어부터는 대문사를 사용한다.(Camel Case)

> ex: `removeSomthing`, `groupNumber`


### 상수
상수(`static final`)는 `Snake Case`를 대문자로 바꾼 형식으로 사용한다.

> ex: `NEGATIVE_INFINITY`

### 지역변수
지역변수에서는 약어를 써도 변수가 사용되는 문맥에서 의미를 쉽게 유추할 수 있기 떄문에 약어를 사용해도 좋다.

그 외에는 다른 멤버와 비슷한 명명 규칙을 적용한다.

### 입력 매개변수
입력 매개변수도 지역변수의 하나이지만, 메서드 설명 문서에까지 등장하기 떄문에 일반 지역변수보다는 신경을 써서 명명하자.

### 타입 매개변수
보통 한 문자로 표현한다.

- 임의의 타입 : `T`
- 컬렉션 원소의 타입 : `E`
- 맵의 키와 값 : `K`, `V`
- 예외 : `X`
- 메서드 반환 타입 : `R`
- 그 외 임의 타입 시퀀스 : `T`, `U`, `V` or `T1`, `T2`, `T3`

---

## 문법 규칙

### 객체를 생성할 수 있는 클래스
단수 명사나 명사구를 사용한다.

> ex: `Thread`, `PriorityQueue`

### 객체를 생성할 수 없는 클래스
복수형 명사로 짓는다.

> ex: `Collectors`, `Collections`

### 인터페이스
클래스와 똑같이 짓거나 `able`, `ible`로 끝나는 형용사로 짓는다.

> ex: `Runnable`, `Iterable`

### 애노테이션
지배적인 규칙없이 명사, 동사, 전치자, 형송사가 두루 쓰인다.

### 메서드
어떤 동작을 수행하는 메서드는 동사나 목적어를 포함한 동사구로 짓는다.

> ex: `append`, `drawImage`

`boolean`값을 반환하는 메서드라면 `is` 또는 `has`로 시작하고 명사나 명사구, 혹은 형송사로 기능하는 아무 단어나 구로 끝나도록 짓는다.

> ex: `isDigit`, `isEmpty`

반환 타입이 `boolean`이 아니거나 해당 인스턴스의 속성을 반환하는 메서드는 명사, 명사구, `get`으로 시작하는 동사구로 짓는다.

> ex: `size`, `getTime`

객체의 타입을 바꿔 다른 타입의 객체를 반환하는 메서드는 보통 `toType`의 형태로 짓는다.

> ex: `toArray`

객체의 내용을 다른 뷰로 보여주는 메서드는 `asType`의 형태로 짓는다.

> ex: `asList`

정적 팩토리 메서드는 `from`, `of`, `instance`, `getInstance` 등의 이름을 흔히 사용한다.
- 더 자세한 내용은 [정적 팩토리 메서드](https://tecoble.techcourse.co.kr/post/2020-05-26-static-factory-method/) 자료를 참고하라


