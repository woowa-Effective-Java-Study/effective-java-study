# 아이템 26 - 로 타입

## 제네릭
클래스나 메소드에서 사용할 내부 데이터 타입을 외부에서 지정하는 기법
   - 제네릭 타입 : 제네릭 클래스, 제네릭 인터페이스, 제네릭 메소드
   - 제네릭 기법의 적용 : 타입 매개변수를 사용한다.
 <br>

## 타입 매개변수
제네릭 기법에서 타입을 변수로 표현하는 것
   - 타입 매개변수의 일반적인 관례
      - 한 문자로 이름을 짓는다.
      - 대문자로 이름을 짓는다.
      - ex) T, E, K, V
```java
class Box<T> { ... }
List<E> cars = new ArrayList<>();
Map<K, V> coins = new HashMap<>();
```
<br>

## 로 타입
타입 파라미터가 없는 제네릭 타입
```java
Collection<E> tickets = ... ; // 제네릭 타입
Collection tickets = ... ; // 로 타입(제네릭이 등장하기 전 컬렉션 선언 방식)
```
  - 제네릭 개념이 등장하기 전 코드와의 호환성을 위해 사용되는 개념
<br>

## 제네릭 타입 vs 로 타입
로 타입을 쓰면 제네릭이 안겨주는 안전성과 표현력을 모두 잃게 된다. 즉 로 타입은 제네릭의 이점을 활용할 수 없다.
### 제네릭을 사용하는 이유
1. 컴파일 타임에서의 강한 타입 체크(타입 안정성)
   - 컴파일 타임에 타입 오류에 대한 검증을 수행함으로써 런타임에는 자료형에 대한 안정성을 보장할 수 있다. 
   - 컴파일 타임 타입 체크가 바람직한 이유 : 컴파일 타임에서 발생한 오류는 런타임 시점에서 발생한 오류보다 더 빠른 시점에 더 쉽게 잡아낼 수 있다.
2. 타입 변환 제거
   - 타입 변환에 들어가는 노력을 줄일 수 있다.
   - 형 변환이 필요 없으므로 가독성이 좋아진다.
 ### 코드 예시
 #### 1. 로 타입 사용
 ```java
 class Neo {}
class Jason {}
class DailyTeam {
    private Object coach;

    public DailyTeam(Object coach) {
        this.coach = coach;
    }

    public Object getCoach()) {
        return coach;
    }
}
```
```java
public static void main(String[] args) {
    DailyTeam neoTeam = new DailyTeam(new Neo());
    DailyTeam jasonTeam = new DailyTeam(new Jason());

    Neo neo = (Neo) neoTeam.getCoach();
    Jason jason = (Jason) neoTeam.getCoach(); // 런타임 에러
}
```
 
#### 2. 제네릭 타입 사용
 ```java
 class Neo {}
class Jason {}
class DailyTeam<T> {
    private T coach;

    public DailyTeam(T coach) {
        this.coach = coach;
    }

    public T getCoach()) {
        return coach;
    }
}
```
```java
public static void main(String[] args) {
    DailyTeam<Neo> neoTeam = new DailyTeam<>(new Neo());
    DailyTeam<Jason> jasonTeam = new DailyTeam<>(new Jason());

    Neo neo = neoTeam.getCoach();
    Jason jason = neoTeam.getCoach(); // 컴파일 에러
}
```
<br>

## 비한정적 와일드카드 타입
어떤 타입이라도 담을 수 있는 가장 범용적인 매개변수화 타입
  - 타입 매개변수를 신경쓰지 않고 선언할 수 있다.
  - 아무 원소나 넣을 수 있으므로 타입 불변식을 훼손할 수 있는 로 타입과 달리 비한정적 와일드카드 타입은 null 이외의 어떤 원소도 넣을 수 없기 때문에 타입 불변식 훼손을 방지할 수 있다.
```java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    unsafeAdd(strings, Integer.valueOf(42));
}

private static void unsafeAdd(List list, Object o) {
    list.add(o); // 런타임 에러
}
```
```java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    unsafeAdd(strings, Integer.valueOf(42));
}

private static void unsafeAdd(List<?> list, Object o) {
    list.add(o); // 컴파일 에러
}
```
<br>

## List\<Object> vs List\<?> vs List
- List의 매개변수로 모든 하위 타입들을 받고 싶은 경우
  - 제네릭 타입 적용 - **List\<Object>**
  - 제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지는 신경쓰고 싶지 않다 - **List\<?>**
    - 컴파일 타임에 원소 삽입이 제한되기 때문에 안전성이 보장된다.
  - 타입 매개변수가 뭐인지도 관심 없고 제네릭 타입도 안 쓸 거다 - **List**
    - 안전성 X
    - 아무 원소나 넣을 수 있기 때문에 타입 불변식이 훼손될 수 있다.
<br>

## 로 타입을 써야하는(허용하는) 예외적인 상황
1. class 리터럴
```java
List.class
String[].class
int.class

List<String>.class // 비허용
List<?>.class // 비허용
```
2. instanceof
  - 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자 사용 시 제네릭 타입을 쓸 필요 X
