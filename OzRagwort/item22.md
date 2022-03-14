# item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

> 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할만을 하자!

## 잘못된 예시 - 상수 인터페이스

상수 인터페이스는 상수를 의미하는 static final 필드로만 가득 찬 인터페이스를 말한다. 

```java
public interface PhysicalConstants {
    static final double AVOGADROS_NUMBER = 6.022140857e23;
    static final double BOLTZMANN_CONSTANT = 1.38064852e-23;
    static final double doubleELECTRON_MASS = 9.10938356e-31;
}
// 사용 예시 1
public class Foo {
    public void foo() {
        double avogadrosNumber = PhysicalConstants.AVOGADROS_NUMBER;
    }
}
// 사용 예시 2
public class Bar implements PhysicalConstants {
    public void foo() {
        double avogadrosNumber = AVOGADROS_NUMBER;
    }
}
```

이 상수를 사용하려는 클래스에서 정규화된 이름을 쓰는 걸 피하고자 상수 인터페이스를 직접 구현하여 사용하기도 한다.

하지만 이렇게 작성하면 클래스 내부에서 사용하는 상수가 외부 인터페이스가 아니라 내부 구현에 해당한다.

이 내부 구현을 노출하는 행위인데 이것은 클래스 사용자에게 혼란을 주기도 하고 사용자의 코드가 이 상수에 종속되게 한다.

심할 경우 이 상수를 쓰지 않아도 바이너리 호환성 때문에 여전히 상수를 인터페이스에 구현해두어야한다.

결국 이 인터페이스를 쓰는 모든 하위 클래스가 이 상수를 가지고있게 된다.

## 바이너리 호환성이란?

인터페이스에 새로운 메서드를 추가했을 때 하위 클래스에 메서드를 추가하지 않아도 호출되기 전까지 기존 클래스 파일이 잘 동작한다는 의미이다.

하지만 그 메서드를 호출하게 되면 런타임 에러가 발생한다.

비슷한 개념으로 소스 호환성, 동작 호환성이라는 개념도 있다.

이것을 바이너리 호환성이라고 한다.

이 문제를 해결하기 위해 만들어진것이 디폴트 메서드다.

디폴트 메서드로 만들면 모든 인터페이스에서 재정의하지 않고 사용할 수 있기 때문에 바이너리 호환성 문제가 발생하지 않는다.

디폴트 메서드는 인터페이스 하위 모든 클래스에서 동작하고 재정의도 할 수 있기 때문에 코드 중복 제거 등의 장점도 있다.

하지만 다중 상속을 하면서 같은 디폴트 메서드가 있을때의 우선순위 문제, 구현 코드 노출 등 문제점도 있다.

## 우선순위 문제

아래의 코드는 어떤 문제가 있을까?

### 문제가 되는 케이스
```java
interface Interface1 {
    default void print() {
        System.out.println("딸기");
    }
}
interface Interface2 {
    default void print() {
        System.out.println("바나나");
    }
}

class ImplementClass implements Interface1, Interface2 {
    // ...
}

public class Foo {
    public void foo() {
        ImplementClass implementClass = new ImplementClass();
        implementClass.print();
    }
}
```

#### 디폴트 메서드 다중 상속 시 우선순위 문제는 다음 규칙을 지키면 된다.

일반적인 우선순위 : 하위 클래스 > 서브 인터페이스 > 가장 상위의 인터페이스

가장 우선순위가 높은 것은 다중 상속 받은 클래스에서 재정의하는 것이다.

두번째로는 extends한 서브 인터페이스이다.

가장 낮은 우선순위는 가장 상위에 인터페이스이다.

상하위 관계가 없는 인터페이스라면 클래스에서 명시적으로 오버라이드해야한다.

### 해결 방법 1 (클래스에서 재정의)
```java
interface Interface1 {
    default void print() {
        System.out.println("딸기");
    }
}
interface Interface2 {
    default void print() {
        System.out.println("바나나");
    }
}

class ImplementClass implements Interface1, Interface2 {
    @Override
    public void print() {
        System.out.println("귤");
    }
}

public class Foo {
    public void foo() {
        ImplementClass implementClass = new ImplementClass();
        implementClass.print();
    }
}
```

### 해결 방법 2 (상하 관계가 명확)
```java
interface Interface1 {
    default void print() {
        System.out.println("딸기");
    }
}
interface Interface2 extends Interface1 {
    default void print() {
        System.out.println("바나나");
    }
}

class ImplementClass implements Interface1, Interface2 {
}

public class Foo {
    public void foo() {
        ImplementClass implementClass = new ImplementClass();
        implementClass.print();
    }
}
```

## 참고 자료
[디폴트 메서드](https://samba-java.tistory.com/22)
[(자바 8) 바이너리 호환성, 소스 호환성, 동작 호환성](http://clearpal7.blogspot.com/2017/03/8_56.html)
[What is binary compatibility in Java?(https://stackoverflow.com/questions/14973380/what-is-binary-compatibility-in-java)
[Java: Binary Compatibility - more than meets the eye](http://codefhtagn.blogspot.com/2010/11/java-binary-compatibility-more-than.html)
