# 아이템 15 - 클래스와 멤버의 접근 권한 최소화

## 정보 은닉
다른 객체에게 자신의 정보를 숨기고 자신의 연산만을 통해 접근을 허용하는 것

### 정보 은닉의 장점
1. 시스템 개발 속도, 재사용성 상승
2. 시스템 관리 비용 절감
3. 성능 최적화 - 직접적인 성능 최적화는 아니지만 다른 컴포넌트에 영향을 주지 않고 필요한 컴포넌트만 성능을 최적화할 수 있다는 점에서 성능 최적화에 도움이 된다.

### 자바에서 제공하는 정보 은닉 장치 - 접근 제어 메커니즘
클래스, 인터페이스, 멤버의 접근성을 명시함으로써 정보 은닉을 구현한다.

<br>

## 접근 제어 메커니즘
- 각 요소의 접근성은 그 요소가 선언된 위치와 접근 제한자로 정해진다.

### 기본 원칙
- 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다. (가장 낮은 접근 수준 부여) 

### 인터페이스와 Top Level 클래스의 접근 수준
- 인터페이스와 Top Level 클래스에 부여할 수 있는 접근 수준 : public, package-private
- public으로 선언하면 공개 API, 내부 구현 용도로만 사용된다면 package-private
- public일 필요가 없는 클래스의 접근 수준은 package-private으로 좁히는 게 바람직

### 멤버의 접근 수준
- 멤버(필드, 메소드, 중첩 클래스, 중첩 인터페이스)에 부여할 수 있는 접근 수준 : private, package-private(default), protected, public
- 멤버 접근성은 최대한 private에 가깝도록 좁히는 게 바람직

### 멤버 접근 수준 제약사항
- 리스코프 치환 원칙 : 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다.
-> 상위 클래스의 메소드를 재정의할 때는 그 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없다.

### 접근 제어 메커니즘 사용 시 권장 사항
- 테스트만을 위해 접근 수준을 넓히는 건 바람직하지 않다.
- public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.
  - 클래스에 public 가변 필드가 존재하면 클래스는 필드에 담을 수 있는 값을 제한할 힘과 스레드 안전성을 잃게 된다.
  - 예외) 상수화(public static final) - 클래스가 표현하는 추상 개념을 완성하는 데 꼭 필요한 구성요소로써의 정적 필드는 상수화를 통해 공개해도 좋다. 이때 해당 필드는 반드시 기본 타입 값이나 불변 객체를 참조해야 한다.
  - 길이가 0이 아닌 배열은 모두 변경 가능하니 public 배열 필드 혹은 배열 필드를 반환하는 메소드의 선언은 지양해야 한다.
```java
private static final Member MEMBER_VALUES = {...};
public static final List<Member> MEMBERS = 
	Collections.unmodifiableList(Arrays.asList(MEMBER_VALUES));
```
```java
private static final Member MEMBER_VALUES = {...};
public static final Member[] members () {
	return MEMBER_VALUES.clone();
}
```
