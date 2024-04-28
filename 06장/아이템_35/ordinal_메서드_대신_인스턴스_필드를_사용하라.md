# 아이템 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

## 열거 타입의 ordinal 메서드

- Enum은 해당 상수가 몇 번째 위치인지를 반환하는 `ordinal()` 메서드를 제공한다.
- `ordinal`을 잘못 사용한 예
    
    ```java
    // 잘못된 버전!
    public enum Ensemble {
        SOLO, DUET, TRIO, QUARTET, QUINTET,
        SEXTET, SEPTET, OCTET, NONET, DECTET;
        
        public int numberOfMusicians() { return ordinal() + 1; }
    }
    ```
    
    - 8명이 연주하는 복4중주를 추가하고 싶다면? 8중주가 이미 있으니 추가할 수 없다.
    - 12명이 연주하는 3중 4중주를 추가하고 싶다면? 11명짜리 dummy 상수를 추가해야 한다.
- 개선된 버전
    
    ```java
    public enum Ensemble {
        SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
        SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
        NONET(9), DECTET(10), TRIPLE_QUARTET(12);
        
        private final int numberOfMusicians;
        Ensemble(int size) { this.numberOfMusicians = size; }
        public int numberOfMusicians() { return numberOfMusicians; }
    }
    ```
    

## 결론

- `ordinal`은 쓰지 말자. 필요하다면 인스턴스 필드로 넣자.
    - [자바 공식 문서](https://docs.oracle.com/javase%2F7%2Fdocs%2Fapi%2F%2F/java/lang/Enum.html#ordinal()): Most programmers will have no use for this method. It is designed for use by sophisticated enum-based data structures, such as `EnumSet` and `EnumMap`.
