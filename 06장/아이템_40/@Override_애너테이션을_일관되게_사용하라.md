  # 아이템40 | @Override 애너테이션을 일관되게 사용하라

  # @Override

- 자바가 기본으로 제공하는 애너테이션
- 메서드 선언에만 달 수 있음
- 애너테이션이 달렸다는 것은 상위 타입의 메서드를 재정의했다는 것을 의미함

<br>

# @Override를 사용하지 않은 경우

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram bigram) {
        return bigram.first == this.first &&
                bigram.second == this.second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }

        System.out.println(s.size()); //260
    }
}
```

Set<Bigram>에 new Bigram(a,a), new Bigram(b,c), … ,  new Bigram(z,z)를 10번 삽입하는 상황

- equals, hashcode를 정의하여 first와 second가 서로 같으면 같은 객체로 인식되기를 기대
- Set<Bigram>의 크기가 26이길 예측했지만 260이 출력됨

→ equals를 **재정의**(Overriding)한 것이 아닌 **다중정의**(Overloading)한 것!

<br>

```java
@Override
public boolean equals(Bigram bigram) {
    return bigram.first == this.first &&
        bigram.second == this.second;
}
```

- 상위 클래스의 메서드 시그니처와 다르기 때문에 컴파일 오류가 발생

```java
@Override
public boolean equals(Object o) {
    if(!(o instanceof Bigram)) {
        return false;
    }
    Bigram b = (Bigram) o;
    return b.first == this.first &&
        b.second == this.second;
}
```
<br>

### 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override를 달자.

- 예외 : 구체 클래스에서 상위 클래스의 추상 메서드를 재정의 할 때는 굳이 @Override를 달지 않아도 된다.
    - 재정의하지 않은 추상 메서드가 있다면 컴파일러가 알려주기 때문

<br>

### 인터페이스의 메서드를 재정의할 때도 @Override를 사용할 수 있다.

- 인터페이스 메서드를 구현한 메서드에도 @Override를 붙이면 시그니처가 올바른지 재차 확신할 수 있음

<br>

### 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 @Override를 다는 것이 좋다.

- 실수로 추가한 메서드가 없다는 것을 보장함

<br>

# 결론

- 재정의한 모든 메서드에 @Override를 의식적으로 달면 우리가 실수했을 때 컴파일러가 알려주니 @Override를 적극 사용하자.
