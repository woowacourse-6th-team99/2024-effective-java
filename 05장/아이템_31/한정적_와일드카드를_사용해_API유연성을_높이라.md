# 아이템 31 : 한정적 와일드카드를 사용해 API 유연성을 높이라

## 핵심정리
- 제네릭의 불공변은 타입안전성을 제공하는 대신, 유연하지 않은 측면이 있다
- 한정적 와일드카드 타입은 제네릭 API의 유연성을 제공한다
- PECS : Producer -Extends / Consumer - Super
- 퍼블 API & 하나의 제네릭 타입 => 비한정적 와일드카드 타입을 사용하자

---
### 문제의식 : 제네릭의 불공변

- List<Type1>은 List<Type2>의 하위 타입도 상위 타입도 아니다

```
[불공변 납득하기]
리스코프 치환원칙 : 하위 타입이 상위 타입의 역할을 대체 가능하다
ex)
- List<Object> & List<String>
- List<Object> : 어떤 객체든 넣을 수 있다
- List<String> : String 객체만 넣을 수 있다
- List<String>은 List<Object>의 역할을 대체 불가하다
=> 리스코프 치환원칙에 위반한다
=> List<String>은 List<Object>;의 하위타입이 아니다
```

그러나, 이러한 불공변의 특징은 어떤 측면에서 비유연성을 제공한다.

예를 들어 다음 같은 Stack 코드가 있다고 가정해보자

### 문제 상황1

```java
    public void pushAll(Iterable<E> src) {
        for (E e : src)
            push(e);
    }

    public static void main(String[] args) {
        Stack<Number> numberStack = new Stack<>();
        Iterable<Integer> integers = Arrays.asList(3, 1, 4, 1, 5, 9);
        numberStack.pushAll(integers); // Iterable<Number>가 아니라 컴파일 되지 않는다
    }
```
- 위의 코드는 컴파일 되지 않는다
- pushAll(Iterable<Number> numbers) 와 Iterable<Integer>는 다른 타입이기 때문이다.

### 문제 상황2
```java
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}

    public static void main(String[] args) {
        Collection<Object> objects = new ArrayList<>();
        numberStack.popAll(objects); // Collection <Number>가 아니라 컴파일 되지 않는다
    }
```
- 위의 코드는 컴파일 되지 않는다
- pushAll(Collection<Number> dst) 와 Collection<Object>는 분명 다른 타입이기 때문이다.

## 해결책 : 한정적 와일드카드 타입

## PECS 원칙
```
PECS : Producer-Extends / Consumer - Super
```
예시를 통해 이해해보자

### Producer -> Extends

```java
    public void pushAll(Iterable<? extends E> src) {
        for (E e : src)
            push(e);
    }

    public static void main(String[] args) {
        Stack<Number> numberStack = new Stack<>();
        Iterable<Integer> integers = Arrays.asList(3, 1, 4, 1, 5, 9);
        numberStack.pushAll(integers); // Integer extends Number 이기에 컴파일 된다
    }
```
- pushAll의 src를 보자
- src는 Stack이 사용할 E 인스턴스를 `생산`한다 => Producer
- 따라서, extends를 통해 E 타입의 하위 타입을 받을 수 있도록 Iterable<? extends E> 로 한정적 와일드 카드 타입을 사용하면 메서드 유연성이 증가한다.

### Consumer -> Super

```java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}

public static void main(String[] args) {
    Collection<Object> objects = new ArrayList<>();
    numberStack.popAll(objects); // Object super Number 이기에 컴파일된다
}
```
- popAll의 dst를 보자
- dst는 Stack으로부터 E 인스턴스를 `소비`한다 => Consumer
- 따라서, super를 통해 E 타입의 상위 타입이 올 수 있도록 Collection<? super E> 를 사용하면 메서드 유연성이 증가한다

<details>
<summary>PECS 원칙 쉽게 외우기</summary>

### Producer와 Consumer의 화살표 방향 설정하기
- Producer와 Consumer를 함수형 인터페이스 Supplier와 Consumer로 생각해보자
- Supplier는 매개변수를 받지 않고 T를 반환한다. => `Supplier ----> T`
- Consumer는 T 타입의 매개변수를 받는다. => `T ----> Consumer`

### 업캐스팅 생각하기
- java에서 업캐스팅은 동적 메서드 탐색을 위해 자연스럽다
- 이에 반해 다운 캐스팅은 강제 형변환이 아닌 이상 자동으로 처리되지 않는다
- 유연한 API라는 것은 객체지향에 자연스러운 업캐스팅에 알맞은 API를 구성하는 것이다

### 화살표 방향 수직화하기
- Supplier와 Consumer를 업캐스팅 방향에 맞게 수직화해보자
- ```
  Consumer
    ^
    T
    ^
  Supplier
  ```
- Supplier가 유연하기 위해서는 T의 하위 타입이어야 한다 => Producer - Extends
- Consumer가 유연하기 위해서는 T의 상위 타입이어야 한다 => Consumer - Super
- 여기서 유연함은 `업캐스팅에 자연스러움`을 의미한다
</details>

<details>
<summary>실습으로 익히는 PECS</summary>

### 문제1. 다음 상황에서 알맞은 한정적 타입 매개변수는?(아이템28)
```java
public class Chooser<T> {
    private final List<T> choiceList;
    private final Random rnd = new Random();

    public Chooser(Collection<?> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```
- Collection<?> choices가 T 타입의 값을 생산
- 정답 : Collection<? extends T>

### 문제2. 다음 상황에서 알맞은 한정적 타입 매개변수는?(아이템30)
```java
public class Union {
    public static <E> Set<E> union(Set<E> s1,
                                   Set<E> s2) {
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }
}
```
- Set<E> s1 과 Set<E> s2가 E 타입의 값을 생산
- 정답 : Set<? extends E> s1, Set<? extends E> s2
</details>

---
## Comparable에 한정적 타입 매개변수 적용해보기

- 다음 public 인터페이스를 가진 max 메서드에 주목하자
- ```java
  public static <E extends Comparable<E>> E max(List<E> list)
  ```

- 먼저 매개변수 List < E > list를 생각하자
- list는 최대값 산출의 자원을 제공(생산)한다. => Producer - Extends
- 따라서 List<? extends E> 가 자연스럽다


그러나 여기서 유연성 강화는 끝나지 않는다

- 만약 `비교 클래스에는 Comparable이 구현되어 있지 않고, 비교하는 클래스의 사우이 클래스에 Comparable이 구현되어 있다면 어떻게 될까?`

```java
public class Box<T extends Comparable<T>> implements Comparable<Box<T>> {

    protected T value;

    public Box(T value) {
        this.value = value;
    }

    public void change(T value) {
        this.value = value;
    }

    @Override
    public int compareTo(Box anotherBox) {
        return this.value.compareTo((T)anotherBox.value);
    }
}

public class IntegerBox extends Box<Integer> {

    private final String message;

    public IntegerBox(int value, String message) {
        super(value);
        this.message = message;
    }
}
```

- 이 경우 IntegerBox는 분명 Box의 일종으로 상위 타입의 규칙을 활용해 비교할 수 있다
- 그러나, Comparable < IntegerBox > 가 구현되어 있지 않다.
- 따라서, 제네릭의 불공변적 성격으로 인해 IntegerBox간의 비교는 성립되지 않는다.
- 즉, Comparable을 직접 구현하지 않고, 구현한 타입을 확장한 타입을 지원하기 위해 와일드 카드가 필요하다

최종적인 max의 인터페이스는 다음과 같다

- ```java
  public static <E extends Comparable<? super E>> E max(List<? extends E> list)
  ```

---

## 비한정적 와일드카드 타입을 통한 유연성

- 타입 매개변수와 와일드카드는 둘중 어느것을 사용해도 괜찮을 때가 많다

```java
// 방안1. 타입 매개변수
public static <E> void swap(List<E> list, int i, int j);

// 방안2. 와일드 카드
public static void swap(List<?> list, int i, int j);
```
- <b>메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하라.</b>
- 신경써야할 타입 매개변수가 없으며 단지 리스트를 넘기면 된다

그러나, 이것에도 문제가 있다
- `비한정적 매개변수는 null을 제외한 어떤 값도 넣을 수 없다`
```java
public static void swap(List<?> list, int i, int j) {
        list.set(i, list.set(j, list.get(i)));
    }
```

- 따라서 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 작성하여 활용하는 것이 좋다
```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
        list.set(i, list.set(j, list.get(i)));
}
```





