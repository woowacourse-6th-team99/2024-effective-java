# **아이템12 | toString을 항상 재정의하라**


# Object의 기본 toString 메서드

- 클래스_이름@16진수로_표시한_해시코드를 반환

```java
// ex) domain.Name@2e7b06
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

<br>

# toString의 일반 규약

- 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환하라
- 모든 하위 클래스에서 재정의하라

<br>

# toString을 재정의해야 하는 이유

toString을  잘 구현한 클래스는 사용하기 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.

<br>

# toString 재정의 시 주의할 점

## 1. 실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다.

```java
public class User {
    private String username;
    private String email;
    private int age;
    private String address;
    
    //age, address의 정보도 포함시키는 것이 좋다.
    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", email='" + email + '\'' +
                '}';
    } 
}
```

만약 객체의 상태가 거대하거나 문자열로 표현하기 적합하지 않는다면 요약정보를 담자

- ex) 맨해튼 거주자 전화번호부(총 123456개)

<br>

## 2. 포맷을 명시하든 아니든 의도를 명확히 밝혀라

### 포맷을 명시하는 경우

```java
/**
* 이 전화번호의 문자열 표현을 반환한다.
* 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
* XXX는 지역코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
* 각각의 대문자는 10진수 숫자 하나를 나타낸다.
* 
* 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
* 앞에서부터 0으로 채워나간다. 예컨데 가입자 번호가 123이라면
* 전화번호의 마지막 네 문자는 "0123"이 된다.
*/
@Override 
public String toString() {
    return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
}
```

장점

- 객체가 표준적이고, 명확하고, 사람이 읽을 수 있게 된다.
- 그 값 자체로 입출력에 사용하거나 사람이 읽을 수 있는 객체로 저장이 가능하다.

단점

- 포맷을 한번 명시하면 평생 그 포맷에 얽매이게 된다.
- 포맷을 수정하는 경우 이를 사용하던 모든 코드와 데이터를 수정해야 한다.

<br>

### 포맷을 명시하지 않는 경우

```java
/**
* 이 약물에 관한 대략적인 설명을 반환한다.
* 다음은 이 설명의 일반적인 형태이나,
* 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
* 
* "[약물 #9: 유형-사랑, 냄새=테러빈유, 겉모습=먹물]"
*/
@Override
public String toString() { ... }
```

장점

- 정책 변경 시 정보를 더 넣거나 포맷을 개선할 수 있는 유연성을 가진다.

단점

- 명확한 포맷이 없어 프로그래머가 임의의 포맷을 정해야 한다.

<br>

# toString이 반환한 값에 포함된 정보를 얻어올 수 있는 Api를 제공하자

toString에 포함된 정보를 가져올 수 있는 api가 없다면 정보가 필요한 프로그래머는 toString을 파싱해서 사용해야 한다.

<br>

# toString이 필요 없는 클래스

- 정적 유틸리티 클래스
- 대부분의 열거 타입
    - 이미 자바가 완벽한 toString을 제공
