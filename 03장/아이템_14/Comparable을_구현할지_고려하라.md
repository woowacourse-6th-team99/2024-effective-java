# 아이템 14. Comparable을 구현할지 고려하라

---

## 핵심 정리

- Comparable 인터페이스 구현 : 순서를 고려해야 하는 값 클래스의 경우 필요
- 정렬, 검색, 비교 기능을 제공하는 컬렉션과 어우러진다
- compareTo 메서드에서 필드의 값을 비교할 때
    - 지양 : 비교연산자(>,<)나 뺄셈을 통한 결과값 산출을 지양하라
    - 권장 : 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드
    - 권장 : Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하라

---

## compareTo와 equals의 차이점

- 동치성 비교에 더해 순서까지 비교가능하다
- 제네릭 하다

<details>
<summary> 제네릭 하다? </summary>

제네릭(Generic)은 `일반적인`이라는 뜻으로 데이터 형식에 의존하지 않고, 하나의 값이 여러 다른 데이터 타입을 가질 수 있도록 하는 방법을 이야기한다.

- equals의 경우, 인수가 Object 타입으로 고정되어 있어, instanceOf으로 타입 확인을 한후 다운 캐스팅이 필수적이다.

```java
    public boolean equals(Object obj){
        return(this==obj);
        }
```

- Comparable은 제네릭 타입을 통해 자료타입에 구애받지 않고 같은 기능을 제공한다.

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

이렇듯 비교할 타입을 컴파일 타임에 정하는 Comparable의 제네릭한 특성은 다음과 같은 이점이 있다.

- 입력 인수의 타입을 확인하거나 형번환할 필요가 없다
- 인수의 타입이 잘못되었다면 컴파일 자체가 되지 않는다

</details>

---

### Comparable의 구현 == 비교를 활용하는 클래스와의 입장권

- Comaparable의 구현은 인스턴스에 순서를 부여하는 것이다
- Comparable을 활용한 수많은 알고리즘과 컬렉션이 있다
- Comparable의 구현은 검색, 극단값 계산, 자동정렬되는 컬렉션 관리 등 비교를 활용하는 클래스와 어울리게 한다

ex) `TreeSet : Comparable 구현의 장점을 보여주는 사례`

```java
public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```

- String이 Comparable을 구현한 덕분에 알파벳 순으로 출력

---

### compareTo의 규약 : 대칭성, 추이성, 반사성

> sgn(표현식) 표기는 부호함수를 뜻하며 값이 음수, 0, 양수일 때 -1,0,1을 반환하도록 정의
<br><br>

1. **대칭성**
  - sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다
  - 따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질 때에 한해 예외를 던져야 한다

2. **추이성**
  - x.compareTo(y) > 0 이고 y.compareTo(z) > 0 =>  `x.compareTo(z) > 0`

3. **반사성**
  - x.compareTo(y)==0 =>  `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`

4. 권고사안
  - x.comapreTo(y) == 0 == x.equals(y)여야 한다. 
  - Comaprable을 구현하고 이 권고를 지키지 않았다면 그 사실을 명시해야 한다.

<details>
<summary> 왜 이런 권고사안이 나왔을까?</summary>

> equals의 결과와 comapreTo의 결과가 일관되지 않으면?<br>   

이 클래스의 객체를 정렬된 클래스에 넣으면 해당 컬렉션이 구현한 인터페이스(Collection, Set,Map)에 정의된 동작과 엇박자를 낼 것이다
 
그 이유는 다음과 같다
- Collection framework는 대부분 equals 규약을 따른다   
- 그러나 정렬된 컬렉션은 동치성을 비교할 때 compareTo를 사용한다

> ex) Collection java 문서 중 일부

``` 
Many methods in Collections Framework interfaces are defined in
terms of the {@link Object#equals(Object) equals} method. 

For example, the specification for the 
{@link #contains(Object) contains(Object o)}
``` 
<br>

> ex) 정렬 컬렉션 : TreeSet java 문서의 일부
```
Note that the ordering maintained by a set must be consistent with equals
if it is to correctly implement the {@code Set} interface.  

This is so because the {@code Set} interface is defined in
terms of the {@code equals} operation, but a {@code TreeSet} instance
performs all element comparisons using its {@code compareTo} (or {@code compare}) method
```
</details>

<details>
<summary>equals != compareTo에서 발생하는 문제를 보여주는 예시</summary>

```java
public final class Example {

    public Set using_Equals_Set = new HashSet<>();
    public Set using_CompareTo_Set = new TreeSet<>();

    public static void main(String[] args) {
        Example example = new Example();
        example.addSet();

        System.out.println(example.using_Equals_Set.size());
        System.out.println(example.using_CompareTo_Set.size());
    }
    public void addSet() {
        using_Equals_Set.add(new BigDecimal("1.0"));
        using_Equals_Set.add(new BigDecimal("1.00"));

        using_CompareTo_Set.add(new BigDecimal("1.0"));
        using_CompareTo_Set.add(new BigDecimal("1.00"));
    }
}
```
- using_Equals_Set : 2개
- using_CompareTo_Set : 1개
- => 사용자 입장에서는 예측가능하지 못한 코드가 될 수 있음
</details>

---

### Comparable 사용법

> **Case1 : 객체 참조 필드가 하나뿐일 때**

ex)
```java
public final class CaseInsensitiveString
        implements Comparable<CaseInsensitiveString> {
  public int compareTo(CaseInsensitiveString cis) {
    return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
  }
}
```

- Comparable &lt; CaseInsensitiveString &gt; : CaseInsensitiveString 참조와만 비교할 수 있다
<br><br>

> % 주의사항 %   
> compareTo 메서드에서 관계연산자(&gt;, &lt;)를 사용하는 방식은 **거추장스럽고 오류를 유발하니 이제는 추천하지 않는다.**

---

> **Case2 : 객체 참조 필드가 여러개일 때**
- 가장 핵심적인 필드부터 비교해나감
- 비교결과가 0이 아니라면(순서가 결정되면) return
- 순서가 결정되지 않으면 똑같지 않은 필드를 찾을 때까지 다음 우선수위의 필드를 비교
<br><br>
#### 방법1 : 순차적인 비교방식
```java
public final class PhoneNumber implements Cloneable, Comparable<PhoneNumber> {
    private final short areaCode, prefix, lineNum; // 우선순위 : areaCode > prefix > lineNum

    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode); //가장 중요한 필드
        if (result == 0) {
            result = Short.compare(prefix, pn.prefix); // 두번째로 중요한 필드
            if (result == 0)
                result = Short.compare(lineNum, pn.lineNum); // 세번째로 중요한 필드
        }
        return result;
    }
}
```
---

#### 방법2 :  Comparator의 메서드 체이닝 활용
- 자바 8부터는 Comparator 인터페이스를 메서드 체이닝을 통해 구현할 수 있게 되었다
- Comparator를 Comparable의 compareTo를 구현하는데 멋지게 활용할 수 있다
- 굉장히 단순하지만 vs 약간의 성능저하가 있다

```java
private static final Comparator<PhoneNumber> COMPARATOR =
        comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
            return COMPARATOR.compare(this, pn);
        }
```

- comparing( 키 추출 함수 ) : 키를 기준으로 순서를 정하는 비교자를 반환
- thenComparing( 키 추출 함수 ) :  Comparator 인스턴스의 메서드, 키 추출자 함수를 입력받아 다시 비교를 반환
- => thenComparing 은 첫번째 비교자를 적용한 다음 새로 추출한 키로 추가비교를 수행한다

<details>
<summary>Comparator의 보조 생성 메서드</summary>

> 자바의 숫자용 기본 타입
- comparintInt / thenComparingInt
- comparingDouble / thenComparingDouble
- comparingLong / thenComparingLong
<br><br>
> 객체 참조용 비교자 생성자 메서드 : comparing / thenComparing
- comparing(keyExtractor) : 키 추출자를 받아 자연적 순서를 사용
- comparing(keyExtractor, Comparator) : 키 추출자를 받아 두번째 인수인 비교자를 사용
<br><br>
- thenComparing(Comparator) : 인수로 받은 비교자로 부차순서를 정함
- thenComparing(keyExtractor) : 키추출하여 키의 자연적 순서를 사용
- thenComparing(keyExtractor, Comparator) : 키 추출자를 받아 두번째 인수인 비교자를 사용
</details>

---

### 값의 차로 순서를 정하는 행위를 주의하라
```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2){
            return o1.hashCode() -o2.hashCode();
        }
}
```
- 정수 오버플로 / 부동 소수점에 따른 오류 발생 가능
- 성능도 그다지 좋지 않다

<br>

### 개선된 코드 1 : 정적 compare 메서드
```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2){
            return Integer.compare(o1.hashCode(), o2.hashCode());
        }
}
```
<br><br>
### 개선된 코드 2 : 비교자 생성 메서드
```java
static Comparator<Object> hashCodeOrder 
        = Comparator.comparingInt(o-> o.hashCode());
}
```
