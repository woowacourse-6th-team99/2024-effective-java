# 아이템36 | 비트 필드 대신 EnumSet을 사용하라

<br>

# 비트 필드 열거 상수 - 구닥다리

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    //비트 필드
    private int style;

    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int styles) { ... }
}
```

- 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있다.

    → 이 집합을 비트 필드라 한다.

```java
text.applyStyles(BOLD | ITALIC); // BOLD와 ITALIC의 OR 비트 연산 -> 01 | 10 = 11(2)
```


<br>

## 비트 연산

### 장점

- 비트별 연산을 통해 합집합과 교집합과 같은 집합 연산을 효율적으로 수행 가능하다.

### 단점

- 정수 열거 상수의 단점을 모두 가진다.
- 비트 필드 값을 해석하기 어렵다.
- 모든 원소를 순회하기 까다롭다.
- 최대 몇 비트가 필요한지 미리 예측하여 적절한 타입(보통 int, long)을 선택해야 한다.

<br>

# EnumSet - 비트 필드를 대체하는 현대적 기법

```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet 이 가장 좋다.
    public void applyStyles(Set<Style> styles) { }
}
```

- 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현한다.
- Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 어떤 Set 구현체와도 함께 사용 가능하다.
- 내부적으로 비트 벡터로 구현되어 비트를 직접 다루지 않고도 효율적으로 처리할 수 있다.

```java
Text text = new Text();
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

<br>

cf ) applyStyles 메서드가 Set<Style>을 받은 이유

- 모든 클라이언트가 EnumSet을 건네리라 짐작되는 상황이라도 이왕이면 인터페이스를 받아라(아이템 64)

→ 이렇게 하면 좀 특이한 클라이언트가 다른 Set 구현체를 넘기더라도 처리할 수 있다.
