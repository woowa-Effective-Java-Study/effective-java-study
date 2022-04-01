# 아이템 30 - 제네릭 메소드

## 제네릭 메소드
클래스와 마찬가지로 메소드도 제네릭으로 만들 수 있다.

#### 타입 안전하지 않은 코드
```java
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1); // raw type
  result.addAll(s2);
  return result;
}
```
#### 제네릭 메소드를 적용한 코드
```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet<>(s1);
  result.addAll(s2);
  return result;
}
```
- 타입 매개변수 목록(\<E>)은 메소드의 제한자와 반환 타입 사이에 위치
- 직접적인 형변환을 해주지 않아도 에러나 경고 없이 컴파일 된다.
- 타입 안전성을 지켜준다.
<br>

## 제네릭 싱글턴 팩토리
불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때 사용하는 패턴으로 제네릭 메소드를 활용하여 구현한다.
```java
public class GenericFactoryMethod { 
    public static final Set EMPTY_SET = new HashSet(); 
  
    public static final <T> Set<T> emptySet() { 
      return (Set<T>) EMPTY_SET; 
    } 
}
```
- 제네릭으로 타입 설정 가능한 인스턴스를 만들어두고, 반환 시에 제네릭으로 받은 타입을 이용해 타입을 결정한다.
```java
@Test 
public void genericTest() { 
    Set<String> set = GenericFactoryMethod.emptySet(); 
    Set<Integer> set2 = GenericFactoryMethod.emptySet(); 
    Set<Elvis> set3 = GenericFactoryMethod.emptySet(); 
    
    set.add("ab"); 
    set2.add(123); 
    set3.add(Elvis.INSTANCE); 
 }
```
- 여러 타입으로 내부 객체를 받아도 에러가 발생하지 않는다.
- 제네릭 메소드를 활용하여 큰 유연성을 제공한다.
<br>
	
## 제네릭 싱글턴 패턴을 적용한 항등함수 클래스
- 항등함수 : f(x) = x
- 제네릭 싱글턴 팩토리를 활용하면 중복 구현 없이 여러 타입을 받을 수 있다.
```java
private static UnaryOperator<Object> IDNTITY_FN = (t) -> t;

@SuppressWarnings("unchecked") // @SuppressWarning로 형변환 경고 숨김
private static <T> UnaryOperator<T> identityFunction() {
		return (UnaryOperator<T>) IDNTITY_FN; // UnaryOperator<Object>를 UnaryOperator<T>로 형변환하는 과정에서 경고 발생
}
```
- 항등함수 객체는 상태가 없기 때문에 요청할 때마다 새로 생성할 필요가 없으므로 static으로 선언한다.
- 항등함수는 입력 값을 수정 없이 그대로 반환하는 특별한 함수이므로 T가 어떤 타입이든 타입 안전성 보장 -> @SuppressWarning로 형변환 경고를 숨겨줘도 문제 없이 동작한다.
#### 제네릭 싱글턴을 적용한 항등함수 클래스를 UnaryOperator\<String>과 UnaryOperator\<Number>로 사용하기
```java
@Test
void genericSingletonEx() {
    String[] strings = {"알린", "알렉스", "배카라"};
    UnaryOperator<String> sameString = identityFunction();
    for (String s : strings) {
        System.out.println(sameString.apply(s));
    }

    Number[] numbers = {1, 2.0, 3L};
    UnaryOperator<Number> sameNumber = identityFunction();
    for (Number n : numbers) {
        System.out.println(sameNumber.apply(n));
    }
}
```
- 형변환을 따로 해주지 않아도 컴파일 오류나 경고가 발생하지 않는다.
<br>
    
## 재귀적 타입 한정
제네릭 메소드에 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정한다.

### Comparable 인터페이스와 재귀적 타입 한정
- 재귀적 한정 타입은 주로 Comparable 인터페이스와 쓰인다.
```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```
- Comparable 인터페이스의 타입 매개변수 T는 Comparable\<T>를 구현한 타입이 자신과 비교할 수 있는 원소의 타입을 정의한다.
    - 거의 모든 타입은 자신과 같은 타입의 원소만 비교할 수 있다.
    - ex) String은 Comparable\<String>을 구현하고 Integer는 Comparable\<Integer>를 구현한다.
- Comparable 기능을 수행하려면 컬렉션에 담긴 모든 원소끼리 상호 비교될 수 있어야 한다는 제약이 존재하는데, 이러한 제약을 재귀적 타입 한정을 이용해 표현할 수 있다.
    ```java
    public static <E extends Comparable<E>> E max(Collection<E> c);
    ```
    - `<E extends Comparable<E>>` : 타입 매개변수의 범위를 Comparable\<E>를 구현한 E만 허용하도록 제한함으로써 "모든 타입 E는 자신과 비교할 수 있다"는 제약을 표현할 수 있다.
    
<br>
  
#### 참고 사이트
- https://jake-seo-dev.tistory.com/13?category=909023 (UnaryOperator)
