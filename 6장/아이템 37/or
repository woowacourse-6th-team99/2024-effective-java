# 아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라

## Plant class
```java
public class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL } // 한해살이, 여러해살이, 두해살이
	
    final String name;
    final LifeCycle lifeCycle;
    
    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;

    @Override public String toString() {
        return name;
    }
}
```
  
## ordinal()을 배열 인덱스로 사용 - 따라 하지 말 것!  
```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycle.length; i++) 
    plantsByLifeCycle[i] = new HashSet<>();

for (Plant p : garden)
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

//결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++; {
    System.out.printf("%s: %s%n",
        Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```
- 비검사 형변환을 수행해야한다. (제네릭과 배열 호환 문제)
- 정확한 정숫값을 사용한다는 걸 직접 보증해야 한다.

  
## EnumMap으로 데이터와 열거 타입을 매핑한다.
```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new enumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lc, new HashSet<>());
for (Plant p : garden) {
    plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
```
- 성능 차이가 크게 나지 않는다.(내부에서 배열을 사용하기 때문)
- 비검사 형변환도 사용하지 않고, 인덱스 계산 과정에서 오류가 날 가능성도 없다!

  
### 스트림을 사용하면 코드를 더 줄일 수 있다.
```java
Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class), toSet()));
```

  
## 배열 인덱스로 ordinal()을 사용 - 따라 하지 말 것!
```java
public enum Phase {
    SOLID, LIQUID, GAS;
	
    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT
		
        private static final Transition[][] TRANSITIONS = {
            { null, MELT, SUBLIME },
            { FREEZE, null, BOIL },
            { DEPOSIT, CONDENSE, null }
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```
- 열거 순서에 의존하기 때문에, 잘못 수정하면 오류가 발생한다! (오작동, 런타임 오류 등)
- 추가되는 요소에 따라 배열도 제곱으로 늘어난다.

  
## EnumMap을 사용하는 편이 훨씬 낫다
```java
public enum Phase {
    SOLID, LIQUID, GAS;
	
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }
		
        //상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>>
            m = Stream.of(values()).collect(groupingBy(
                t -> t.from, toMap(t -> t.to, t -> t, (x, y) -> y, () -> new EnumMap<>(Phase.class))));
				
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```
