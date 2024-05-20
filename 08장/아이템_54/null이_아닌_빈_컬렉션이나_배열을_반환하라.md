# 아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

### 좋지 않은 예시

```java
// 리스트가 비어 있으면 null을 반환하는 코드
private final List<Cheese> cheeses;

public List<Cheese> getCheeses() {
    return cheeses.isEmpty() ? null : new ArrayList<>(cheese);
}

// 그러면 다음과 같이 null 처리 코드가 추가되어야 한다.
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.MOZZARELLA))
    System.out.println("모짜렐라 치즈를 찾았습니다.");
```

- 항상 null 방어 코드를 작성해야 하는 단점이 있다.
- 방어 코드를 빼먹으면 오류가 발생할 수 있으며, 컬렉션이 비어 있는 경우가 드물면, 수년 뒤에 발견될 수 있다.

### 아니 빈 컨테이너 만드는 것도 비용인데 그냥 null 반환하면 안 됨? 🤔🤔

1. 그 정도 성능 저하는 웬만하면 신경 쓸 수준이 아니다.
2. 굳이 새로 안 만들어도 됨.
    
    ```java
    // 좋은 예시
    public List<Cheese> getCheeses() {
        return new ArrayList<>(cheese);
    }
    ```
    
3. 빈 컬렉션 할당이 (예를 들면) 1000만 번 일어나면 성능이 떨어질 수 있음.
    1. 그럴 때는 매번 똑같은 빈 불변 컬렉션을 반환하자.
    2. `Collections.emptyList` 같은 메서드를 활용하면 됨.
    
    ```java
    // 좋은 예시
    public List<Cheese> getCheeses() {
        return cheeses.isEmpty() ? Collections.emptyList()
            : new ArrayList<>(cheese);
    }
    ```
    

### 배열은 어떻게 씀?

- 길이가 0인 배열을 올바르게 반환하는 방법
    
    ```java
    public Cheese[] getCheeses() {
        return cheeses.toArray(new Cheese[0]);
    }
    ```
    
- 성능 저하가 우려되는 경우 다음과 같이 쓰자.
    
    ```java
    public static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
    
    public Cheese[] getCheeses() {
        return cheeses.toArray(EMPTY_CHEESE_ARRAY);
    }
    ```
    

# 결론

- null이 아닌, 빈 배열이나 컬렉션을 반환하라.
- null을 반환하는 API는 오류 처리 코드가 늘어나고, 성능이 유의미하게 좋지도 않다.
