# 📚📚 이펙티브 자바 3판(Effective Java 3/E)
* 저자 : [조슈아 블로크](https://github.com/jbloch)
* 예제 코드
  - [한글판](https://github.com/WegraLee/effective-java-3e-source-code)
  - [원본](https://github.com/jbloch/effective-java-3e-source-code)

## Ch2. 객체 생성과 파괴
|                        아이템                         |                                                 정리 링크                                                 |
|:--------------------------------------------------:|:-----------------------------------------------------------------------------------------------------:|
|         **아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라**         | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch02/Item1.md) |
|         **아이템 2. 생성자 매개변수가 많다면 빌더를 고려하라**          | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch02/Item2.md) |
|     **아이템 3. private 생성자나 열거 타입으로 싱글톤임을 보증하라**     | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch02/Item3.md) |
|     **아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라**      | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch02/Item4.md) |
|      **아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라**      | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch02/Item5.md) |
|             **아이템 6. 불필요한 객체 생성을 피하라**             | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch02/Item6.md) |
|             **아이템 7. 다 쓴 객체 참조를 해제하라**             | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch02/Item7.md) |
|       **아이템 8. finalizer와 cleaner의 사용을 피하라**       | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch02/Item8.md) |
| **아이템 9. try-finally보다는 try-with-resources를 사용하라** | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch02/Item9.md) |

## Ch3. 모든 객체의 공통 메서드
|                     아이템                     |                                                 정리 링크                                                  |
|:-------------------------------------------:|:------------------------------------------------------------------------------------------------------:|
|     **아이템 10. equals는 일반 규약을 지켜 재정의하라**     | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch03/Item10.md) |
| **아이템 11. equals를 재정의하려거든 hashCode도 재정의하라** | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch03/Item11.md) |
|       **아이템 12. toString을 항상 재정의하라**        | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch03/Item12.md) |
|      **아이템 13. clone 재정의는 주의해서 진행하라**       | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch03/Item13.md) |
|      **아이템 14. Comparable을 구현할지 고려하라**      | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch03/Item14.md) |

## Ch4. 클래스와 인터페이스
|                          아이템                           |                                                 정리 링크                                                  |
|:------------------------------------------------------:|:------------------------------------------------------------------------------------------------------:|
|           **아이템 15 : 클래스와 멤버의 접근 권한을 최소화하라**           | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/Effective%20Java%203E/Ch04/Item15.md) |
| **아이템 16 : public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라** | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch04/Item16.md) |
|               **아이템 17 : 변경 가능성을 최소화하라**               | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch04/Item17.md) |
|             **아이템 18 : 상속보다는 컴포지션을 사용하라**              | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch04/Item18.md) |
|   **아이템 19 : 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라**   | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch04/Item19.md) |
|           **아이템 20 : 추상 클래스보다는 인터페이스를 우선하라**           | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch04/Item20.md) |
|          **아이템 21 : 인터페이스는 구현하는 쪽을 생각해 설계하라**          | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch04/Item21.md) |
|         **아이템 22 : 인터페이스는 타입을 정의하는 용도로만 사용하라**         | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch04/Item22.md) |
|        **아이템 23 : 태그 달린 클래스보다는 클래스 계층구조를 활용하라**        | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch04/Item23.md) |
|         **아이템 24 : 멤버 클래스는 되도록 static으로 만들라**          | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch04/Item24.md) |
|          **아이템 25 : 톱레벨 클래스는 한 파일에 하나만 담으라**           | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch04/Item25.md) |

## Ch5. 제네릭
| 아이템                          | 정리 링크                                                                                                  |
|------------------------------|--------------------------------------------------------------------------------------------------------|
| **아이템 26 : 로 타입은 사용하지 말라**   | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch05/Item26.md) |
| **아이템 27 : 비검사 경고를 제거하라**    | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch05/Item27.md) |
| **아이템 28 : 배열보다는 리스트를 사용하라** | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch05/Item28.md) |

## Ch6. 열거 타입과 애너테이션

| 아이템                                           | 정리 링크                                                                                                  |
|-----------------------------------------------|--------------------------------------------------------------------------------------------------------|
| **아이템 34 : int 상수 대신 열거 타입을 사용하라**            | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch06/Item34.md) |
| **아이템 35 : ordinal 메서드 대신 인스턴스 필드를 사용하라**     | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch06/Item35.md) |
| **아이템 36 : 비트 필드 대신 EnumSet을 사용하라**           | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch06/Item36.md) |
| **아이템 37 : ordinal 인덱싱 대신 EnumMap을 사용하라**     | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch06/Item37.md) |
| **아이템 38 : 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라** | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch06/Item38.md) |
| **아이템 40 : @Override 애너테이션을 일관되게 사용하라**       | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch06/Item40.md) |
| **아이템 41 : 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라**     | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch06/Item41.md) |


## Ch7. 람다와 스트림

| 아이템                             | 정리 링크                                                                                                  |
|---------------------------------|--------------------------------------------------------------------------------------------------------|
| **아이템 42 : 익명 클래스보다는 람다를 사용하라** | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch07/Item42.md) |
| **아이템 43 : 람다보다는 메서드 참조를 사용하라** | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/Effective%20Java%203E/Ch07/Item43.md) |


<br>
<br>

--------------------------------

<br>
<br>

# 📚📚 읽기 좋은 코드가 좋은 코드다
* 저자 : 더스틴 보즈웰, 트레퍼 파우커

## 서론
|           챕터           |                                                   정리 링크                                                    |
|:----------------------:|:----------------------------------------------------------------------------------------------------------:|
| **1. 코드는 이해하기 쉬워야 한다** | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/ch01.md) |

## PART 1
|          챕터           |                                                        정리 링크                                                        |
|:---------------------:|:-------------------------------------------------------------------------------------------------------------------:|
|   **2. 이름에 정보 담기**    | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/Part%201/ch02.md) |
|  **3. 오해할 수 없는 이름들**  | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/Part%201/ch03.md) |
|       **4. 미학**       | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/Part%201/ch04.md) |
|  **5. 주석에 담아야하는 대상**  | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/Part%201/ch05.md) |
| **6. 명확하고 간결한 주석 달기** | [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/main/The%20Art%20of%20Readable%20Code/Part%201/ch06.md) |

## PART 2
|          챕터           |                                                        정리 링크                                                        |
|:---------------------:|:-------------------------------------------------------------------------------------------------------------------:|
| **7. 읽기 쉽게 흐름제어 만들기** | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/Part%202/ch07.md) |
| **8. 거대한 표현을 잘게 쪼개기** | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/Part%202/ch08.md) |
|    **9. 변수와 가독성**     | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/Part%202/ch09.md) |

## PART 3
|           챕터           |                                                        정리 링크                                                        |
|:----------------------:|:-------------------------------------------------------------------------------------------------------------------:|
| **10. 상관없는 하위문제 추출하기** | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/Part%203/ch10.md) |
|    **11. 한번에 하나씩**     | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/Part%203/ch11.md) |
|  **12. 생각을 코드로 만들기**   | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/Part%203/ch12.md) |
|    **13. 코드 분량줄이기**    | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/Part%203/ch13.md) |

## PART 4
| 챕터               | 정리 링크                                                                                                               |
|------------------|---------------------------------------------------------------------------------------------------------------------|
| **14. 테스트와 가독성** | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/The%20Art%20of%20Readable%20Code/Part%204/ch14.md) |


<br>
<br>

--------------------------------

<br>
<br>

# 📚📚 HTTP 완벽가이드
* 저자 : 데이빗 고울리, 브라이언 토티, 마조리 세이어, 세일루 레디, 안슈 아가왈

## 1. HTTP : 웹의 기초
|        챕터        |                                                                                            정리 링크                                                                                            |
|:----------------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
|  **4. 커넥션 관리**   | [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/HTTP%20%EC%99%84%EB%B2%BD%20%EA%B0%80%EC%9D%B4%EB%93%9C/4%EC%9E%A5.%20%EC%BB%A4%EB%84%A5%EC%85%98%20%EA%B4%80%EB%A6%AC.md) |
|   **9. 웹 로봇**    |                                 [링크](https://github.com/MinChul-Son/Book-for-Developer/tree/readme/HTTP%20%EC%99%84%EB%B2%BD%20%EA%B0%80%EC%9D%B4%EB%93%9C)                                 |
| **10. HTTP/2.0** |                   [링크](https://github.com/MinChul-Son/Book-for-Developer/blob/main/HTTP%20%EC%99%84%EB%B2%BD%20%EA%B0%80%EC%9D%B4%EB%93%9C/10%EC%9E%A5.%20HTTP%202.0.md)                    |

