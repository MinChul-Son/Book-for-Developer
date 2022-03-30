# 아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

`Collection`이나 `Array`가 비었을 때 `null`을 반환한다면 클라이언트는 항시 `null`을 체크하는 방어 코드가 필요하다.

### 빈 `Collection`이나 `Array`를 반환하는 것에는 비용이 들어 `null`이 좋다?
이는 사실이 아니다.

빈 `Collection`과 `Array`를 할당하는 것이 성능에 차이를 줄 만큼의 수준이 아니다.

굳이 새로 할당하지 않고도 반환할 수 있다.

`Collection.emptyList`, `Collection.emptySet`, `Collection.emptyMap`을 사용해 새로 할당하지 않고 반환할 수 있다.

빈 배열을 반환할 때는 길이가 0인 배열을 반환하자.

매번 할당하는 것이 성능 저하에 문제가 될 것 같다면 미리 할당해두고 반환하도록 하자.

```
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```
