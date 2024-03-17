# 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라

## hashCode를 재정의하지 않았을 때의 문제점

- HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제가 생긴다.
    
    ```java
    Map<PhoneNumber, String> m = new HashMap<>();
    m.put(new PhoneNumber(707, 867, 5309), "제니");
    m.get(new PhoneNumber(707, 867, 5309)); // null 반환
    ```
    

## hashCode 규약

- `equals` 비교에 사용되는 정보가 변경되지 않았다면, **항상 같은 값을 반환**해야 한다.
- `equals(Object)`가 두 객체를 같다고 판단했다면, `hashCode`도 같아야 한다.
- `equals(Object)`가 두 객체를 다르다고 판단했더라도 `hashCode`가 같을 필요는 없다.
하지만 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

## 항상 같은 hashCode를 반환할 때의 문제점

```java
// 사용 금지
@Override
public int hashCode() {
    return 42;
}
```

- 모든 객체가 해시테이블의 버킷 하나에 담겨 linked list처럼 동작함.
- 평균 수행 시간이 O(1)인 해시테이블이 O(n)으로 느려짐.

## ⚠️ 주의사항

- 파생 필드(다른 필드로부터 계산해낼 수 있는 필드)는 해시코드 계산에서 제외해도 된다.
- `equals` 비교에 사용되지 않은 필드는 **반드시 제외**해야 한다.
- 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.
- `hashCode`가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.
  - 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.

## 결론

- `hashCode`를 재정의하지 않으면 `HashMap`이나 `HashSet`을 사용할 때 문제를 일으킨다.
- 논리적으로 다른 객체면 `hashCode`도 다르게 하자.
- AutoValue 프레임워크, Lombok(`@EqualsAndHashCode`), IDE가 좋은 `hashCode`를 만들어준다.
