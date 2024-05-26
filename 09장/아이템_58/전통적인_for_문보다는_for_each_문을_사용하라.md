# 아이템 58. 전통적인 for문보다는 for-each 문을 사용하라

### for 문을 사용하는 코드

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); } {
    Element e = i.next();
    // ...
}
```

```java
for (int i = 0; i < a.length; i++) {
    // ...
}
```

- 쓸데 없는 iterator와 인덱스 변수 때문에 코드가 더럽다.
- 변수를 잘못 사용할 경우 문제가 생길 수 있다. (컴파일러로 잡기도 어렵다)
- 컬렉션과 배열의 코드 형태가 많이 다르다.

### for-each 문(= enhanced for statement)으로 개선한 코드

```java
for (Element e : elements) {
    // ...
}
```

- 반복 대상이 컬렉션이든 배열이든 동일하게 사용할 수 있다.
- 성능도 그대로이다.

### for 문에서의 버그 발생

```java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX } // 주사위의 눈

Collection<Face> faces = EnumSet.allOf(Face.class);

for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
    for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
        System.out.println(i.next() + " " + j.next());
```

- 36개 조합이 나올 것을 예상했지만, `ONE ONE`부터 `SIX SIX`까지 6개의 조합만 나온다.

### for 문으로 버그 해결

```java
for (Iterator<Face> i = faces.iterator(); i.hasNext(); ) {
    Face face = i.next();
    for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
        System.out.println(face + " " + j.next());
}
```

- 일단 원하는 대로 돌아가기는 한다.
- 근데 코드가 보기 안 좋다.

### for-each 문으로 코드 개선

```java
for (Face face1 : faces)
    for (Face face2 : faces)
        System.out.println(face1 + " " + face2);
```

- 코드가 훨씬 간결해졌다.

### for-each 문을 사용할 수 없는 상황

1. 파괴적인 필터링(destructive filtering)
    - 컬렉션을 순회하면서 선택된 원소를 제거해야 하는 경우에는 for-each 사용 불가
    - `Collection`의 `removeIf`를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있음
    
    ```java
    List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
    
    // 런타임 에러
    for (int num : numbers) {
        numbers.remove(num);
    }
    ```
    
2. 변형(transforming)
    - 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 index 사용 불가피
    
    ```java
    List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
    
    // 에러는 안 뜨는데 값이 바뀌지 않는다.
    for (int num : numbers) {
        num = 4;
    }
    ```
    
3. 병렬 반복
    - 여러 컬렉션을 병렬로 순회해야 하는 경우, 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

### (심화) for-each를 사용하려면 `Iterable` 인터페이스를 구현해야 한다.

- 원소들의 묶음을 표현하는 타입을 작성해야 한다면 `Iterable` 구현을 고민해 보자.
- for-each를 사용할 수 있어서 유리하다.
