# 아이템 13. clone 재정의는 주의해서 진행하라

## Object.clone()과 Cloneable 인터페이스
```java
protected native Object clone() throws CloneNotSupportedException;
```

```java
public interface Cloneable {}
```

- `clone()`을 사용하려면 Cloneable을 구현해야 한다.
- 그러나, `clone()`은 Object 클래스 내에 작성되어 있다!
- 또한, 접근 제어자가 protected이기 때문에 public으로 `Override` 해서 제공해야 한다.  

###

## Object.clone() 명세
객체의 복사본을 생성해 반환한다. '**복사**'의 의미는 클래스마다 다르다.

일반적인 의도로 작성되었을 때, 다음 식이 참이다.(그러나 반드시 만족하는 것은 아니다.)
```java
x.clone() != x	//true
```
```java
x.clone().getClass() == x.getClass()	//true
```
```java
x.clone().equals(x)	//true
```
###
관례상, 반환할 객체를 `super.clone()`을 호출해 얻어야 한다. 이 관례를 따른다면, 다음 식은 참이다.
```java
x.clone().getClass() == x.getClass()	//true
```
##

### `super.clone()`을 호출해 작성해야 하는 이유
```java
public class A implements Cloneable {

    @Override
    public A clone() {
        return new A();
    }
}
```
- 이를 상속받은 클래스의 `clone()` 메소드는 A 타입 객체를 반환한다!
- 클래스가 final이고 `super.clone()`을 호출하지 않는다면 Cloneable을 구현할 이유가 없다.  
###

### 가변 참조가 없는 클래스용 clone 메서드
```java
@Override
    public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```
- `super.clone()`이 Object를 반환하므로 형변환하여 반환해주자.(절대 실패하지 않는다)
- `super.clone()`이 검사 예외(checked exception)를 던지므로 try-catch 블록으로 감싸야 한다. 
###  

### 가변 상태를 참조하는 클래스용 clone 메소드
```java
@Override
    public Stack clone() {
        try {
            Stack result = (Stack) super.clone(); 
            result.elements = elements.clone();	// private Object[] elements; 
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```
- 내부 정보를 복사하지 않은 Stack 복사본은 같은 Object 배열을 참조한다!
- 필드가 final이라면 새로운 값을 할당할 수 없기 때문에 final 한정자를 제거해야 할 수도 있다.
(`가변 객체를 참조하는 필드는 final로 선언하라`는 일반 용법과 충돌한다)
- 복잡한 가변 객체의 경우, deep copy를 사용하거나 모든 필드를 초기화한 다음 원본 객체 상태를 다시 생성해야 한다.
##

## 더 나은 객체 복사 방식 - 복사 생성자, 복사 팩터리
```java
public Yum(Yum yum) {...};
```
```java
public static Yum newInsatance(Yum yum) {...};
```
- 모순적이고 엉성하게 문서화된 규약에 기대지 않는다.
- 정상적인 final 용법과 충돌하지 않는다.
- 불필요한 검사 예외를 던지지 않는다.
- 형변환도 필요하지 않다.
###

### 인터페이스를 인수로 받는 경우 - 변환 생성자, 변환 팩터리
```java
HashSet<Object> s = new HashSet<>();
TreeSet<Object> t = new TreeSet<>(s);
```
