# 아이템 03. private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 싱글턴(singleton)이란?

- 인스턴스를 오직 하나만 생성할 수 있는 클래스

## 싱글턴을 왜 쓰나요?

- 메모리 낭비를 방지할 수 있음.
    - 요청이 계속 들어올 때마다 계속 new 해주면 객체가 너무 많이 생성됨.
- 싱글턴의 인스턴스는 전역 인스턴스이기 때문에 다른 여러 클래스에서 데이터를 공유해서 사용할 수 있음.

## 싱글턴의 문제점은?

- 너무 많은 일을 하는 경우, 결합도(의존성)가 높아진다.
    - 개방 폐쇄 원칙 위반
    - 개방 폐쇄 원칙(OCP): 확장에 대해서는 Open, 수정에 대해서는 Close
- private 생성자 때문에 상속이 어렵다.
- 테스트하기가 힘들다.
    - 단위 테스트는 독립적이어야 함. 근데 싱글턴은 자원을 공유하기 때문에 독립적이지 못할 수 있음.

## 1. public static final 필드 방식

```java
public class Elvis { 
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    
    public void leaveTheBuilding() { ... }
}
```

### 장점

- 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
- 간결하다.

## 2. 정적 팩터리 방식

```java
public class Elvis { 
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }
    
    public void leaveTheBuilding() { ... }
}
```

### 장점

- 확장성: API를 바꾸지 않고도 싱글턴이 아닌 방식으로 변경할 수 있다.
- 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
    - 제네릭 싱글턴 팩터리: 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리
- 정적 팩터리의 메서드 참조를 Supplier로 사용할 수 있다.
    - Supplier: 인자를 받지 않고 값을 반환하는 함수형 인터페이스
    - `Elvis::getInstance`를 `Supplier<Elvis>`로 사용

## 위의 두 방식의 단점

- 리플렉션 API를 사용하면 private 생성자를 호출할 수 있으므로 막아 주어야 함.
    - 리플렉션(`java.lang.reflect`): 생성자, 메서드, 필드 조작 가능 / private이어도 접근 가능
- 직렬화 → 역직렬화 과정에서 새로운 객체가 생겨 버림. 따라서 이것을 방지하는 로직이 별도로 필요함.
    - 직렬화: 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트 형태로 변환하는 기술
    - 직렬화(serialize): **객체 → byte** 형태로 데이터 변환
    - 역직렬화(deserialize): **byte → 객체** 형태로 데이터 변환

## Enum 방식 — 대부분의 상황에서 바람직한 방법

```java
public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding() { ... }
}
```

### 장점

- public 필드 방식보다 더 간결하다.
- 추가 노력 없이 직렬화할 수 있다.
- 복잡한 직렬화 상황, 리플렉션 공격에도 제2의 인스턴스 생성을 완벽히 막아준다.

### 단점

- 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.

## 결론

- 상속이 필요 없으면 Enum 싱글턴을 쓰자.
- 상속이 필요하면 public 필드 방식을 쓰되, 리플렉션 및 직렬화에서의 취약점에 대비하는 로직을 작성하자.

## 참고한 사이트

- https://jeong-pro.tistory.com/86
- [https://velog.io/@jhbae0420/싱글톤-패턴의-사용-이유와-문제점](https://velog.io/@jhbae0420/%EC%8B%B1%EA%B8%80%ED%86%A4-%ED%8C%A8%ED%84%B4%EC%9D%98-%EC%82%AC%EC%9A%A9-%EC%9D%B4%EC%9C%A0%EC%99%80-%EB%AC%B8%EC%A0%9C%EC%A0%90)
- [https://inpa.tistory.com/entry/JAVA-☕-싱글톤-객체-깨뜨리는-방법-역직렬화-리플렉션](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%8B%B1%EA%B8%80%ED%86%A4-%EA%B0%9D%EC%B2%B4-%EA%B9%A8%EB%9C%A8%EB%A6%AC%EB%8A%94-%EB%B0%A9%EB%B2%95-%EC%97%AD%EC%A7%81%EB%A0%AC%ED%99%94-%EB%A6%AC%ED%94%8C%EB%A0%89%EC%85%98)
