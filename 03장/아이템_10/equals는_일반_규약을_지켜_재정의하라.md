# 아이템 10. equals는 일반 규약을 지켜 재정의하라

equals 메서드는 재정의하기 쉬워 보이지만 곳곳에 함정이 도사리고 있어서 자칫하면 끔찍한 결과를 초래한다. <br>
문제를 회피하는 가장 쉬운 길은 아예 재정의하지 않는 것이다. 다음에서 열거한 상황 중 하나에 해당한다면 재정의하지 않는 것이 최선이다.

**1. 각 인스턴스가 본질적으로 고유하다.**

값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당한다. 예. Thread

**2. 인스턴스의 '논리적 동치성'을 검사할 일이 없다.**

논리적 동치성이 애초에 필요하지 않다고 판단했다면 Object의 기본 equals만으로 해결된다.

**3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.**

대부분의 Set, List, Map 구현체는 AbstractSet, AbstractList, AbstractMap으로부터 상속받아 그대로 쓴다. 

**4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.**

<br>

그렇다면 equals를 재정의해야 할 때는 언제일까? 객체 식별성(두 객체가 물리적으로 같은가)이 아니라 <ins>논리적 동치성을 확인해야 하는데,</ins><br> 상위 클래스의 equals가 논리적 동치성을 비교하도록 <ins>재정의되지 않았을 때다.</ins>

- 주로 값 클래스들이 여기 해당한다.
- 논리적 동치성을 확인하도록 재정의해두면, 값을 비교하길 윈하는 기대에 부응함은 물론 Map의 키와 Set의 원소로 사용할 수 있게 된다. 

<br>

equals 메서드를 재정의할 때는 <ins>반드시 일반 규약을 따라야 한다.</ins> 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다.

- 반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해 x.equals(x)는 true다. 
- 대칭성(symmetry): null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true면 y.equals(x)도 true다.
- 추이성(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다. 
- 일관성(consistency): null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

<br>

Object 명세에서 말하는 동치관계란 무엇일까?
- 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산이다. 
- 이 부분집합을 동치류(equivalence class; 동치 클래스)라 한다. 
- <ins>equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다.</ins>

<br>

## 반사성 

**객체가 자기 자신과 같아야 한다는 뜻이다.** 

이 요건을 어긴 클래스의 인스턴스를 컬렉션에 넣은 다음 contains 메서드를 호출하면 방금 넣은 인스턴스가 없다고 답할 것이다. 

<br>

## 대칭성 

**두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻이다.**

```java
@Override  
public boolean equals(Object o) {  
    if (o instanceof CaseInsensitiveString) {  
        return s.equalsIgnoreCase(  
                ((CaseInsensitiveString) o).s);  
    }  
    if (o instanceof String) {  
        return s.equalsIgnoreCase((String) o);  
    }  
    return false;  
}
```

- CaseInsensitiveString의 equals는 일반 String을 알고 있지만 String의 equals는 CaseInsensitiveString의 존재를 모른다.

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");  
String s = "polish";
```


- `cis.equals(s)`는 true를 반환하고, `s.equals(cis)`는 false를 반환하여 대칭성을 위반한다.

이 문제를 해결하려면 CaseInsensitiveString의 equals를 String과도 연동하겠다는 허황한 꿈을 버려야 한다. 그 결과 equals는 다음처럼 간단한 모습으로 바뀐다.

```java
@Override  
public boolean equals(Object o) {  
    return o instanceof CaseInsensitiveString &&  
            ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);  
}
```

<br>

## 추이성

**첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻이다.**

2차원에서의 점을 표현하는 Point 클래스와 새로운 색상 정보를 필드로 추가한 하위 ColorPoint 클래스를 예로 들어보자.

```java
// 잘못된 코드 - 대칭성 위배!
@Override  
public boolean equals(Object o) {  
    if (!(o instanceof ColorPoint)) {  
        return false;  
    }  
    return super.equals(o) && ((ColorPoint) o).color == color;  
}
```

- Point의 equals는 색상을 무시하고, ColorPoint의 equals는 입력 매개변수의 클래스 종류가 다르다며 매번 false를 반환할 것이다. 

ColorPoint.equals가 Point와 비교할 때는 색상을 무시하도록 하면 해결될까? 이 방식은 대칭성은 지켜주지만, 추이성을 깨버린다. 

```java
// 잘못된 코드 - 추이성 위배!
@Override  
public boolean equals(Object o) {  
    if (!(o instanceof Point)) {  
        return false;  
    }  
    if (!(o instanceof ColorPoint)) { // o가 일반 Point면 색상을 무시하고 비교한다.
        return o.equals(this);  
    }  
    return super.equals(o) && ((ColorPoint) o).color == color;  
}
```

```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);  
Point p2 = new Point(1, 2);  
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

- `p1.equals(p2)`와 `p2.equals(p3)`는 true를 반환하는데, `p1.equals(p3)`가 false를 반환한다.
- 또한, 이 방식은 무한 재귀에 빠질 위험도 있다. 
	- Point의 또 다른 하위 클래스로 SmellPoint를 만들고, equals는 같은 방식으로 구현했다고 해보자. 그런 다음 `myColorPoint.equals(mySmellPoint)`를 호출하면 StackOverflowError를 일으킨다.

그럼 해법은 무엇일까? 사실 이 현상은 모든 객체 지향 언어의 동치관계에서 나타나는 근본적인 문제다. <ins>구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.</ins>

이번 equals는 같은 구현 클래스의 객체와 비교할 때만 true를 반환한다. 괜찮아 보이지만 실제로 활용할 수는 없다.

```java
// 잘못된 코드 - 리스코프 치환 원칙 위배!
@Override  
public boolean equals(Object o) {  
    if (o == null || o.getClass() != getClass()) {  
        return false;  
    }  
    Point p = (Point) o;  
    return p.x == x && p.y == y;  
}
```

- Point의 하위 클래스는 정의상 여전히 Point이므로 어디서든 Point로써 활용될 수 있어야 한다. 그런데 이 방식에서는 그렇지 못하다. 

<br>

구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만 괜찮은 우회 방법이 하나 있다. <ins>상속 대신 컴포지션을 사용하면 된다.</ins>
- Point를 ColorPoint의 private 필드로 두고, ColorPoint와 같은 위치의 일반 Point를 반환하는 뷰 메서드를 public으로 추가하는 식이다. 

<br>

자바 라이브러리에도 구체 클래스를 확장해 값을 추가한 클래스가 종종 있다. 예. Timestamp
- Timestamp를 이렇게 설계한 것은 실수니 절대 따라해서는 안 된다. 
- 추상 클래스의 하위 클래스에서라면 equals 규약을 지키면서도 값을 추가할 수 있다. 상위 클래스를 직접 인스턴스로 만드는 게 불가능하다면 지금까지 이야기한 문제들은 일어나지 않는다. 


<br>

## 일관성 

**두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 뜻이다.**

- 가변 객체는 비교 시점에 따라 서로 다를 수도 혹은 같을 수도 있는 반면, 불변 객체는 한번 다르면 끝까지 달라야 한다. 
- 클래스가 불변이든 가변이든 equal의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다. 
	- 예. URL의 equals는 호스트 이름을 IP 주소로 바꾸려면 네트워크를 통해야 하는데, 그 결과가 항상 같다고 보장할 수 없다. 
	- equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.

<br>

## null-아님

**모든 객체가 null과 같지 않아야 한다는 뜻이다.**

일반 규약은 실수로 NullPointerException을 던지는 코드도 허용하지 않는다. <ins>그러나 명시적 null 검사는 필요 없다.</ins>
- 동치성을 검사하려면 equals는 건네받은 객체를 적절히 형변환한 후 필수 필드들의 값을 알아내야 한다. 
- 그러려면 형변환에 앞서 instanceof 연산자로 입력 매개변수가 올바른 타입인지 검사해야 한다.
- instanceof는 (두 번째 피연산자와 무관하게) 첫 번째 피연산자가 null이면 false를 반환한다.

<br>

---

지금까지의 내용을 종합해서 양질의 equals 메서드 구현 방법을 단계별로 정리해보겠다. 

<br>

**1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.**

**2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.**

**3. 입력을 올바른 타입으로 형변환한다.**

**4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.**

<br>

- float와 double을 제외한 기본 타입 필드는 == 연산자로 비교하고, 참조 타입 필드는 각각 equals 메서드로, float와 double 필드는 각각 정적 메서드인 Float.compare(float, float)와 Double.compare(double, double)로 비교한다. 
	- float와 double을 특별 취급하는 이유는 Float.NaN, -0.0f, 특수한 부동소수 값 등을 다뤄야 하기 때문이다.
	- Float.equals와 Double.equals 메서드는 오토박싱을 수반할 수 있으니 성능상 좋지 않다. 
- null도 정상 값으로 취급하는 참조 타입 필드는 정적 메서드인 Objects.equals(Object, Object)로 비교해 NullPointException 발생을 예방하자
- 앞서의 CaseInsensitiveString 예처럼 비교하기가 아주 복잡한 필드를 가진 클래스는 그 필드의 표준형(canonical form)을 저장해둔 후 표준형끼리 비교하면 훨씬 경제적이다. 
	- 이 기법은 특히 불변 클래스에 제격이다. 가변 객체라면 값이 바뀔 때마다 표준형을 최신 상태로 갱신해줘야 한다. 
- 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하자. 
	- 핵심 필드로부터 계산해낼 수 있는 파생 필드 역시 굳이 비교할 필요는 없지만, 파생 필드를 비교하는 쪽이 더 빠를 때도 있다. 파생 필드가 객체 전체의 상태를 대표하는 상황이 그렇다.
- equals를 재정의할 땐 hashCode도 반드시 재정의하자.
- 일반적으로 별칭(alias)은 비교하지 않는 게 좋다. 
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
	- 이 메서드는 입력 타입이 Object가 아니므로 재정의가 아니라 다중정의(아이템 52)한 것이다.
 
