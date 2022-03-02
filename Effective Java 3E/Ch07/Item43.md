# 아이템 43. 람다보다는 메서드 참조를 사용하라

메서드 참조는 람다보다도 더 간결하게 코드를 만들 수 있다.

```
// 두 가지 모두 하는 일이 동일하다.
(base, exponent) -> Math.pow(base, exponent);

Math::pow;
```

무조건적으로 간결한 것이 읽고 좋은 것은 아니므로 상황에 맞게 판단하자!

<br>

## 메서드 참조 유형

### 1. 정적 메서드를 가리키는 메서드 참조
```
Integer::parseInt

//----------------------

str -> Integer.parseInt(str)
```

### 2. 수신 객체를 특정하는 한정적 인스턴스 메서드 참조
```
Instant.now()::isAfter

//----------------------

Instant then = Instant.now();
t -> then.isAfter(t)
```
정적 참조와 비슷하게 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 같다.

### 3. 수신 객체를 특정하지 않는 비한정적 인스턴스 메서드 참조
```
String::toLowerCase

//---------------------

str -> str.toLowerCase()
```

### 4. 클래스 생성자 메서드 참조
```
TreeMap<K, V>::new

//---------------------

() -> new TreeMap<K, V>()
```

### 5. 배열 생성자 메서드 참조
```
int[]::new

//--------------------

len -> new int[len]
```
