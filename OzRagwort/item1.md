# item 1. 생성자 대신 정적 팩터리 메서드를 고려하라

### 클래스의 인스턴스를 생성하는 방법 두 가지

1. public 생성자

```java
public class Car {
    public Car() {
    }
}
```

2. 정적 팩터리 메서드

```java
public class Car {
    public Car() {
    }

    public static final Car newInstance() {
        // 메서드에 final을 붙이면 상속받았을 때 수정을 못함
        return new Car();
    }
}
```

두 방법 모두 똑같이 새로운 Car를 만들어 반환함.

### 알게모르게 사용하고 있던 정적 팩터리 메서드

특히 Arrays, Collections 등에서 많이 사용됨.

- Arrays.asList()
- Arrays.copyOf()
- Collections.unmodifiableList()
- Collections.unmodifiableMap()
- String.valueOf() 등 원시 값의 VO의 valueOf() 등

### 정적 팩터리 메서드의 특징

static으로 선언되어있기 때문에 객체를 생성하지 않고 사용할 수 있음. 하지만 결국 내부에서는 new Object(...)처럼 객체 인스턴스를 만들어 반환하는 메서드임.

---

# 정적 팩터리의 장점. 왜 정적 팩터리를 사용하는가?

1. 이름을 가질 수 있다.
2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를반환할 수 있다.
5. 정적 팩터리 메서드를 작성하는 시점에서는 반환할 객체의 클래스가 존재하지 않아도 된다.

## 1. 이름을 가질 수 있다.

- 한 클래스에서 여러개의 생성자가 있다면 한눈에 각 생성자가 어떤 동작을 하는 생성자인지 알기 어렵다.

사용자는 파라미터만 보고 이 메서드의 기능을 추측해서 사용해야한다.

```java
public class People {
    public People() {
    }

    public People(String name) {
    }

    public People(String name, int age) {
    }
}
```

이런 경우 생성자가 아닌 정적 팩터리 메서드로 이름을 달아 사용자의 이해를 도울 수 있다. 1번의 경우보다 2번의 경우가 사용자가 이해하기 쉽고 유지보수하기도 쉽다.

```java
public class People {
    public People(String name) {
    } // 1

    public static withName(String name) {
    } //2
}
```

- 같은 형식의 파라미터를 가지는 생성자가 필요할 경우 정적 팩터리로 구분하여 만들 수 있다.

```java
public class People {
    public People(String name) {
    }

    public People(String Address) {
    }
}
```

이렇게 생성자를 만들 경우 에러가 발생한다. 이런 경우 정적 팩터리 메서드를 이용하는 것이 적절하다.

```java
public class People {
    public static withName(String name) {
    }

    public static withAddress(String address) {
    }
}
```

## 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

읽기 전 퀴즈

```text
Integer i1 = Integer.valueOf(1);
Integer i2 = Integer.valueOf(1);
Integer i3 = new Integer(1);
Integer i4 = new Integer(1);
boolean b1 = i1 == i2; // 1
boolean b2 = i1 == i3; // 2
boolean b3 = i1 == i4; // 3
```

b1, b2, b3는 과연 무엇일까요?

고민해보고 다음으로 넘어가기

---

부제 : 불변 클래스를 캐싱할 수 있다.

정적 팩터리 메서드는 static으로 선언되어 있기 때문에 호출을 위해 새로운 인스턴스를 생성하지 않아도 된다.

따라서 불변 클래스를 미리 만들어 캐싱하고 재활용하면 불필요하게 객체 생성을 피할 수 있다.

### Integer.valueOf()에서 불변 클래스 캐싱 알아보기

```java
// 실제 코드와 조금 다릅니다! 자세한 코드가 궁금하다면 직접 Integer 클래스를 까보세요 :)
public class Integer {
    private static class IntegerCache {
        static final int low = -128;
        static final int high = 127;
    }

    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
}
```

Integer.valueOf(1)을 하면 실제로는 이미 만들어서 가지고있는 Integer 객체를 반환한다.

그래서 Integer.valueOf(1)는 늘 같은 불변 객체가 반환된다.

### 플라이웨이트 패턴(Flyweight pattern)

객체 내부에서 Pool을 만들고 객체를 관리하고 있음.

요청이 오면 Pool에 객체가 있으면 그대로 반환, 없으면 생성하여 반환해주는 방법.

```java
public class Number {
    private static final int MAX = 100;
    private static Number[] pool = new Number[MAX]; // static이라 프로그램 종료 전까지 남아있음

    private int number;

    public static valueOf(int number) {
        if (0 <= number && number < MAX) {
            if (pool[number] == null) {
                pool[number] = new Number(number); // 없으면 만들어서 반환
            }
            return pool[number]; // 있으면 바로 반환
        }
    }
}
```

### 인스턴스 통제 클래스(instance-controlled class)

정적 팩터리 메서드로 인스턴스를 만드는 클래스는 인스턴스를 통제할 수 있다.

여기서 통제의 의미는 클래스를 싱글톤으로 만든다는 의미이다.

싱글톤 패턴은 1개의 인스턴스를 오직 한번만 만들 수 있어 단일 인스턴스가 보장되는 것.

예를들면 이런게 싱글톤 패턴이라고 할 수 있는것같다.

```java
public class Boolean {
    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);

    private boolean b;

    public Boolean(boolean b) {
        this.b = b;
    }

    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
}
```

이렇게 통제하면 불변 값 클래스에서 동치인 인스턴스가 자기뿐임을 보장할 수 있다.

a == b여야 a.equals(b)가 된다는 말임

## 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

정적 팩터리 메서드로 인터페이스를 반환하면 구현 클래스를 공개하지 않고도 객체를 만들어 반환할 수 있다.

객체는 구현 클래스의 상세를 숨길 수 있고, 사용자는 객체를 인터페이스로 다룰 수 있다.

아래 코드를 보면 사용자에게 반환되는 데이터는 Animal 타입을 가진다. 그것이 Cat인지 Lion인지는 공개하지 않는다.

```java
public interface Animal {
}

public class Cat implements Animal {
}

public class Lion implements Animal {
}

public class AnimalGenerator {
    public static Animal of(boolean cute) {
        if (cute) {
            return new Cat();
        }
        return new Lion();
    }
}
```

이렇게 클래스를 만들면 구현 클래스를 API에 노출시키지 않아도 되기 때문에 API를 작게 유지할 수 있다.

이런것을 이용하는 예시가 `java.util.Collections`이다.

예시 몇 가지)

- Collections.unmodifiableList()
    - 리턴 : List, 구현 클래스 : UnmodifiableList
- Collections.unmodifiableMap()
    - 리턴 : Map, 구현 클래스 : UnmodifiableMap
- Collections.synchronizedList()
    - 리턴 : List, 구현 클래스 : SynchronizedList
- Collections.singletonList()
    - 리턴 : List, 구현 클래스 : SingletonList
- Collections.emptyList()
    - 리턴 : List, 구현 클래스 : EmptyList

예시는 List지만 Map, Set 등의 메서드도 있다.

## 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

3번의 내용과 어느정도 이어지는 내용이다.

반환하는 데이터의 타입이 반환 타입으로 정해진 타입의 하위면 그냥 반환할 수 있다는 것이다.

```java
public interface MyInt {
    static MyInt of(int v) {
        MyInt instance;

        if (v > 100) {
            instance = new MyBigInt(v);
        } else {
            instance = new MySmallInt(v);
        }

        return instance;
    }

    String getValue();

    class MySmallInt implements MyInt {
        private int value;

        MySmallInt(int v) {
            this.value = v;
        }

        @Override
        public String getValue() {
            return "MySmallInt: " + this.value;
        }
    }

    class MyBigInt implements MyInt {
        private int value;

        MyBigInt(int v) {
            this.value = v;
        }

        @Override
        public String getValue() {
            return "MyBigInt: " + this.value;
        }
    }
}

public class Main {
    public static void main(String[] args) {
        MyInt mySmallInt = MyInt.of(10);
        MyInt myBigInt = MyInt.of(200);

        System.out.println(mySmallInt.getValue());
        System.out.println(myBigInt.getValue());
    }
}
```

[코드 참고](https://velog.io/@lychee/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-%EC%95%84%EC%9D%B4%ED%85%9C-1.-%EC%83%9D%EC%84%B1%EC%9E%90-%EB%8C%80%EC%8B%A0-%EC%A0%95%EC%A0%81-%ED%8C%A9%ED%84%B0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC-%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC#4-%EC%9E%85%EB%A0%A5-%EB%A7%A4%EA%B0%9C%EB%B3%80%EC%88%98%EC%97%90-%EB%94%B0%EB%9D%BC-%EB%A7%A4%EB%B2%88-%EB%8B%A4%EB%A5%B8-%ED%81%B4%EB%9E%98%EC%8A%A4%EC%9D%98-%EA%B0%9D%EC%B2%B4%EB%A5%BC-%EB%B0%98%ED%99%98%ED%95%A0-%EC%88%98-%EC%9E%88%EB%8B%A4)

이 코드를 실행시키면 mySmallInt는 MySmallInt 구현체로, myBigInt는 MyBigInt 구현체로 생성이 되어 반환된다.

하지만 둘다 MyInt의 하위 클래스이기 때문에 제대로 동작한다. 3번이랑 비슷한 내용.

## 5. 정적 팩터리 메서드를 작성하는 시점에서는 반환할 객체의 클래스가 존재하지 않아도 된다.

서비스 제공자 프레임워크를 만드는 근간이 된다고 한다.

서비스 제공자 프레임워크는 세 가지 컴포넌트로 이루어져있다.

1. 서비스 인터페이스(service interface): 구현체의 동작을 정의
2. 제공자 등록 API(provider registration API): 구현체를 등록
3. 서비스 접근 API(service access API): 서비스의 인스턴스를 얻을 때

서비스 제공자 프레임워크는 구현체의 동작을 정의하고 구현체를 등록하고 인스턴스를 얻는 과정으로 되어있다.

정적 팩터리 메서드는 서비스의 인스턴스를 얻을 때 사용하는데 메서드 작성 시점에는 구현체가 정의되어있지 않아도 된다.

왜냐하면 나중에 동작을 정의해서 구현체로 등록하면 되기 때문이다.

이런 예시가 JDBC(Java Database Connectivity)가 있다.

JDBC는 어떤 jdbc를 등록하는지에 따라 `getConnection()`로 반환되는 객체가 다르다.

```text
Class.forName("oracle.jdbc.driver.OracleDriver");
DriverManager.getConnection(url, user, pw); // 정적 팩터리 메서드
```

`getConnection()`의 코드를 작성하는 시점에서는 객체 클래스가 없어도 된다는 것이다.

[JMT 프로그래밍](https://web2eye.tistory.com/220) 의 코드를 참고하여 설명한다.

`newInstance()` 정적 팩터리 메서드를 사용하는 시점에서는 어떤 객체를 사용할지 모른다. path로 주어진 경로에 있는 객체를 사용하게 될 것이다.

```java
public abstract class Company {
    public static Company getInstance(String path) {
        Company company = null;
        try {
            Class<?> childCompany = Class.forName(path);
            company = (Company) childCompany.newInstance();

        } catch (ClassNotFoundException e) {
            System.out.println("클래스가 없습니다.");
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        }
        return company;
    }

    public abstract String getCompanyName();
}
```

Company를 상속한 자식 클래스 Woowahan를 만든다.

```java
public class Woowahan extends Company {
    @Override
    public String getCompanyName() {
        return "Woowahan";
    }
}
```

이후 테스트해보면 결과는 다음과 같다.

```java
public class CompanyTest {
    @Test
    @DisplayName("정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다")
    void canNotReturnObjectClassWhenUseStaticFactoryMethod() {
        Company nhn = Company.getInstance("item1.Woowahan"); // 실제 경로
        assertThat(nhn.getCompanyName()).isEqualTo("Woowahan"); // true

        Company kakao = Company.getInstance("item1s.Woowacourse"); // 존재하지 않는 path
        assertThat(kakao.getCompanyName()).isEqualTo("Woowacourse"); // 에러
    }
}
```

# 정적 팩터리의 단점

## 1. 정적 팩터리 메서드'만' 있으면 하위 클래스를 만들 수 없다.

public, protected 생성자가 없으면 하위 클래스를 못 만든다.

```java
public class Animal {
    private Animal() {
    }

    public static Animal getInstance() {
        return new Cat();
    }
}

public class Cat extends Animal {
    private Cat() {
        super(); // 에러 발생
    }
}
```

하지만 이것은 관점에 따라서 장점이 될 수도 있다고 한다.

상속을 하면 두 객체사이에 의존이 생기는데 이런 의존이 생기지 않도록 방지하기 때문에 장점이 될 수도 있다고한다.

해결하기 위해서는 컴포지션을 사용하면 된다고 한다.

컴포지션이란 상속이 아닌 인스턴스 변수로 가져오는 방법이다.

```java
public class Animal {
    private Animal() {
    }

    public static Animal getInstance() {
        return new Cat();
    }
}

public class Cat {
    private Animal animal;
}
```

## 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

생성자가 아니라 결국 메서드이기 때문에 공식적인 문서등에서 정적 팩터리 메서드를 찾아내기 어렵다.

[11버전 Integer 공식 문서](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html) 로 예를든다.

이 문서에서 생성자는 Constructor Summary로 명시한다. 하지만 정적 팩터리 메서드인 `valueOf()`는 수 많은 Method Summary에 섞여있기 때문에 이것이 정적 팩터리 메서드인지 일반
메서드인지 알기 어렵다.

[11버전 Colletions 공식 문서](https://docs.oracle.com/javase/7/docs/api/java/util/Collections.html) Collections를 이용한 정적 팩터리
메서드를 찾는 것도 매우 어렵다.

Collections는 생성자가 private로 되어있고 수많은 정적 팩터리 메서드가 있지만 sort, add, addAll과 같은 일반 메서드들과 함께 정리되어있어 구분하기 어렵다.

---

# 정적 팩터리 메서드 이름 방식

- from: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드

  `Date d = Date.from(instant);`

- of: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드

  `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`

- instance 혹은 getInstance: 매개변수로 명시한 인스턴스를 반환, 같은 인스턴스임을 보장하진 않는다.

  `StackWalker luke = StackWalker.getInstance(options);`

- create 혹은 newInstance: instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스 생성을 보장

  `Object newArray = Array.newInstance(classObject, arrayLen);`

- getType: getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용

  `FileStore fs = Files.getFileStore(path); ("Type"은 팩터리 메서드가 반환할 객체의 타입)`

- newType: newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용

  `BufferedReader br = Files.newBufferedReader(path);`

- type: getType과 newType의 간결한 버전

  `List<Complaint> litany = Collections.list(legacyLitany);`

---

## 같이 이야기해볼 것

1. 싱글톤 패턴을 왜 써야하고 무슨 장점이 있을까?
2. 구현 클래스를 API에 노출시키지 않아도 되는 것이 왜 장점일까?
3. 정적 팩터리 메서드를 사용했던 경험이 있다면 -> 언제, 왜 사용했는지 이야기 해보자.
