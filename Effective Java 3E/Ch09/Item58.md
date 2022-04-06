# 아이템 58. 전통적인 for문보다는 for-each문을 사용하라

`for-each`문은 반복자와 인덱스 변수를 사용하지 않으므로 코드가 깔끔해지고 오류가 날 일이 없다.

하나의 관용구로 `Collection`과 `Array`를 모두 처리할 수 있기 때문에 어떤 컨테이너를 다루는지를 신경쓰지 않아도 된다.

## Collection 중첩 순회
`Collection`을 중헙 순회해야한다면 `for-each`문의 이점이 더 커진다.

```
for (Suit suit : suits)
    for (Rank rank : ranks)
        deck.add(new Card(suit, rank));
```

## for-each를 사용할 수 없는 상황
- 파괴적인 필터링 : `Collection`을 순회하면서 선택된 원소를 제거할 때(`java 8`부터는 `removeIf`를 사용하여 해결 가능)
```
List<Person> people = new ArrayList<>();

people.add(new Person("민철"));
people.add(new Person("영수"));
people.add(new Person("철수"));

for (Person person : people) {
    if (person.getName().equals("민철") {
        people.remove(person); // -> ConcurrentModificationException 발생!!
    }
}
```

이는 `removeIf`를 사용해 해결할 수 있다.
```
for (Person person : people) {
    people.removeIf(people -> people.getName().equals("민철"));
}
```

- 변형 : 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야하는 상황
```
List<Person> people = new ArrayList<>();

people.add(new Person("민철"));
people.add(new Person("영수"));
people.add(new Person("철수"));

for (Person person : people) {
    if (person.getName().equals("민철") {
        person = new Person("손민철"); // -> ConcurrentModificationException 발생!!
    }
}
```

이 상황에서는 전통적인 `for`문을 사용하면 된다.

```
for (int i = 0; i < people.size()l i++) {
    if (people.get(i).getName.equals("민철") {
        people.set(i, new Person("손민철"));
    }
}
```
- 병렬 반복 : 여러 컬렉션을 병렬로 순회한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야한다.
