### 인스턴스화란? 
<details>
<summary>[개념적 관점] 클래스 / 객체 / 인스턴스의 차이점</summary>
![image](https://github.com/woowacourse-6th-book-study/2024-effective-java/assets/148152234/48153add-6b9c-4062-964b-5ea202d759d6)

**클래스**
- 객체를 생성하는 일종의 설계도
- 붕어빵(인스턴스)를 만들기 위한 일종의 틀
- 속성(필드)와 동작(메서드)로 이루어진다

**객체**
- 자신의 속성을 가지고 있고, 다른 것과 식별가능 한 것

**인스턴스**
- 소프트웨어 내에서 구현한 실체
- 설계도(붕어빵 틀)를 따라 생성된 객체 하나하나에 해당

---
![image](https://github.com/woowacourse-6th-book-study/2024-effective-java/assets/148152234/9844bd2c-1672-482c-88a6-6b8ad0aaf945)

#### [몇가지 짚고 넘어갈 점]

<u>1. 객체는 클래스의 상위개념이다.</u>
- 클래스는 객체를 구현하는 하나의 방식이다.
- 객체를 소프트웨어 세계로 옮기는 방식은 꼭 클래스가 아니어도 된다.
   
   
<u>2. 인스턴스와 객체는 다른 개념이다.</u>
- 객체는 자신 고유의 속성을 가지는 대상이다.
- 그 객체를 구현한 설계도가 클래스이다.
- 그 설계도인 클래스를 통해 메모리에 할당한(실체화) 데이터가 인스턴스이다.

즉, 객체의 속성과 행동을 클래스라는 설계도로 표현하고,   
그 <span style = "color:red">속성과 행동을 가지는 실체적 메모리 개체들이 인스턴스</span>이다.
</details>
<details>

<summary>[CS적 관점] 클래스 / 객체 / 인스턴스의 차이점</summary>
<br>
![image](https://github.com/woowacourse-6th-book-study/2024-effective-java/assets/148152234/4f62737f-d498-48d5-9f7d-3a3a342c469f)

**메서드 영역(Method Area)**
- 메서드에 대한 정보
- 정적 변수/ 메서드에 대한 정보
- 멤버변수 이름, 접근제어자, 타입등 클래스에 대한 정보
<br>
  
**호출스택(Call Stack)**
- 메서드 수행에 필요한 메모리 공간을 할당받고 작업 완료 후 반환
<br>

**힙(Heap)**
- 인스턴스가 생성되는 동적 할당 공간
- 인스턴스 변수를 관리
  ​
  여기서 인스턴스 변수들은 힙영역에서 관리하고  
  ​클래스나 정적필드에 있는 정보들은 메서드 영역에서 관리한다는 사실을 기억하자

---

​그럼 CS 관점에서의 인스턴스화를 해석해보자  
​
```java
Person bro = new Person("bro");
Person coli = new Person("coli");
```
​
Person : <span style="background-color:#fff5b1">클래스 영역</span>에 Person 클래스가 로드된다

new Person() : 생성자로 인스턴스가 생성되어 <span style="background-color:#fff5b1">힙 메모리</span>에 로드된다

coli(참조변수) : <span style="background-color:#fff5b1">스택영역</span>에 로드되어 힙 메모리 영역에 저장된 인스턴스 주소값을 가리킨다

---

여기서 인스턴스화에 대한 인사이트를 가질 수 있다
> * 인스턴스화는 힙 메모리에 인스턴스가 로드되는 과정을 뜻한다.   
> * 인스턴스화는 생성자로 인해 시발된다.   
> * 즉, 생성자를 호출할 수 없다면 인스턴스화가 되지 않는다.

<br>
그렇게 해서 나온 문장이 item4이다

### "인스턴스화를 막으려거든 private 생성자를 사용하라"
</details>

---
## **1. 인스턴스를 막는 방법**
- private 생성자 선언하기

```java
public class UtilityClass{
    
    //기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용)
    private UtilityClass(){
        thorw new AssertionError();
    }
    
}
```

---
### **2. 정적 유틸리티 클래스 : 인스턴스화를 막아야 하는 케이스**

- 인스턴스를 만들어 사용하는 것이 목표가 아닌 클래스들이 존재한다  
- static 메서드와 static 필드만 담을 정적 유틸리티 클래스가 그것이다  

<details>
<summary> 정적 유틸리티 클래스란?</summary>

정적 유틸리티 클래스는 다음 3가지 특징을 포함한다
> * static : 모든 변수와 메서드는 정적(static)이다
> * stateless : 상태를 가지지 않는다(인스턴스 변수를 가지지 않는다)
> * 상속되지 않는다. by final class / private 생성자

```
용도1. 모든 클래스간의 공유할 전역변수를 만들고자 할 때
용도2. 모든 클래스에서 자주 사용되는 Utility 메소드를 모아놓을 때
```
**>> 그럼 이제 정적 유틸리티 클래스가 왜 인스턴스화가 필요없는 클래스인지 알 수 있다**
 - 인스턴스화는 CS관점으로 "힙 영역에 인스턴스라는 메모리를 할당하는 행위" 이다. 
 - 힙영역에서는 인스턴스 변수와 런타임에 변할 수 있는 정보를 담당한다 
 - 정적 유틸리티 클래스가 가진 정보 중 <span style="background-color:#fff5b1">힙영역이 관리할 정보는 존재하지 않는다</span> 
 - => 그렇기에 인스턴스화에 대한 유인이 없다
</details>
<hr>

> **Case1. 기본 타입값이나 관련 정적 메서드를 모아놓고 싶을 때**

- 정적 메서드와 정적 변수만을 제공하는 클래스 
- 특정 기능과 속성의 공유를 목적으로 만들어진다 
- 즉, 각자의 식별자와 상태값을 가지는 인스턴스가 필요하지 않다

`ex) java.util.Arrays : 기본 타입값이나 배열 관련 정적 메서드를 모아놓는다`

``` java
public class Arrays {
    // private 생성자를 통해 인스턴스화를 막아주고 있다
    // 왜? : 공유기능을 명시한 유틸리티 클래스니까
    // Suppresses default constructor, ensuring non-instantiability.
    private Arrays() {}

    public static void sort(int[] a) {}

    public static boolean equals(double[] a, double[] a2) {
        //엄청 엄청난 로직
    }
}
```

<hr>

> **Case2. 정적 팩토리 메서드를 모아놓은 클래스**
- 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아놓는 클래스 
- 팩토리 메서드를 모아놓은 클래스 
- 모두 정적이기 때문에 인스턴스화를 통해 얻는 이득이 없다

ex) `java.util.Collections`

```java
    public class Collections {

    // Suppresses default constructor, ensuring non-instantiability.
    private Collections() {}

    private static final int BINARYSEARCH_THRESHOLD   = 5000;
    private static final int REVERSE_THRESHOLD        =   18;
    private static final int SHUFFLE_THRESHOLD        =    5;
    private static final int FILL_THRESHOLD           =   25;
    private static final int ROTATE_THRESHOLD         =  100;
    private static final int COPY_THRESHOLD           =   10;
    private static final int REPLACEALL_THRESHOLD     =   11;
    private static final int INDEXOFSUBLIST_THRESHOLD =   35;

    public static <T> Collection<T> unmodifiableCollection(Collection<? extends T> c) {}
    public static <T> Set<T> unmodifiableSet(Set<? extends T> s) {}
    public static <T> Collection<T> synchronizedCollection(Collection<T> c) {}
}
```

---

> **Case3. final 클래스와 관련된 메소드를 모아놓은 클래스들**
-  final 클래스는 상속을 통한 오용방지를 위해 상속이 금지된 클래스이다. ex) String 클래스
- 그렇기에 <span style="background-color:#fff5b1">final클래스와 관련한 메서드를 모아놓을 때<span>에도 private 생성자를 넣어주어야 한다.
-  final 클래스를 상속해서 하위 클래스에 메서드를 넣는 것이 불가능하기 때문이다.

<details>

<summary> final 클래스 != private 생성자 </summary>

- 다음 위의 문장은 `final 클래스라고 private 생성자를 넣어주라는 의미가 아니다.` 
- final 클래스의 생성자가 외부에 호출될 필요가 있다면 당연히 생성자를 public으로 열어주어야 한다. 
   

- 예를 들어 사용자가 입력한 이름을 가진 Animal 객체를 상속한 final 클래스 Dog이 있다고 가정해보자
```java
public class Animal {
        private String name;

        public Animal(String name) {
                this.name = name;
        }
}


public final class Dog extends Animal{

    public static void bark(){
        //개 짖는 소리좀 안나게 해라
    }
}
```

여기서 Dog의 <span style="background-color:#fff5b1">생성자를 private으로 막아두면 상위 클래스인 Animal의 name을 받아줄 수 없다.</span>
하위 클래스의 생성은 상위 클래스의 생성이 보장되어야 가능하므로 검사오류가 뜬다
![image](https://github.com/woowacourse-6th-book-study/2024-effective-java/assets/148152234/ed2ea90d-1ff7-4271-a57d-6a989a0a635b)

그럼 여기서 말하는 **인스턴스화가 불필요한 final 클래스**는 무엇일까?
   
문장을 다시보자   
-  final 클래스는 상속을 통한 오용방지를 위해 상속이 금지된 클래스이다. ex) String 클래스
- 그렇기에 final클래스와 관련한 메서드를 모아놓을 때에도 private 생성자를 넣어주어야 한다. 
- final 클래스를 상속해서 하위 클래스에 메서드를 넣는 것이 불가능하기 때문이다.

이는 <span style="background-color:#fff5b1">정적 유틸리티 클래스로의 기능을 하는 final 클래스</span>를 함의한다.

> **해석1. final 클래스는 상속이 금지된 클래스이다**
> final 클래스가 다른 클래스를 상속받지 않고, 그 자체로 존재한다면 더이상 상속되지 못하고 final 클래스는 상위 클래스로의 지위를 갖지 못한다. 이는 하위 클래스가 이 클래스의 생성자를 호출할 일이 없다는 것을 의미한다.

<br>

> **해석2. final 클래스와 관련된 메서드를 모아놓은 클래스**
> 상태가 없음을 함의한다. 즉 공유되는 기능만을 모아놓은 클래스이다

<br>

> **해석3. 상속이 안되므로 오버라이딩 할 수 없다**
> 상속으로 인해 런타임에서 메서드 오버라이딩에 대해 판단하는 상황이 발생하지 않는다
> 즉 메서드가 정적으로 고정되어 있다

그렇기에 이 클래스는 정적 유틸리티 클래스를 함의하는 final 클래스임을 알 수 있다

예를 들어 `jUnit의 StringUtils 클래스`와 같은 정적 유틸리티 클래스를 들 수 있다
![image](https://github.com/woowacourse-6th-book-study/2024-effective-java/assets/148152234/b0088750-2f49-422f-8a51-eee6684eb054)

</details>

---

### **Q. 생성자를 생성하지 않으면?**

> 컴파일러가 자동으로 public 기본 생성자를 생성 => 인스턴스화를 막지 못함
​
---

### **Q . 추상클래스를 만들어 해결하면 되지 않나요?**

추상클래스는 인스턴화를 하지 못한다. (∵ 아직 구현되지 않은 추상메서드)  
그러나, 이 경우에도 private 생성자를 명시해주는 것이 좋다

> 이유1. 하위클래스를 만들어 인스턴스화 가능  
> 이유2. 상속해서 쓰라는 뜻으로 오해가능
