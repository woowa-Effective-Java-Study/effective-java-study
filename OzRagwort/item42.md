# item 42. 익명 클래스보다는 람다를 사용하라

## 함수 객체(function object)

인터페이스의 인스턴스를 나타내는 말이다.

특정 함수나 동작을 나타낼 때 보통 쓴다.

1.1버전 부터 익명 클래스라는 방법으로 함수 객체를 만들고 있다.

예를 들면!

### 익명 클래스를 사용하는 경우

```java
public class Foo {
    public void foo() {
        Arrays.sort(length, new Comparator<int[]>() {
            @Override
            public int compare(int[] o1, int[] o2) {
                return o1[1] - o2[1];
            }
        });
    }
}
```

[백준 - 14247](https://github.com/OzRagwort/ozragwort-algorithms/blob/e42f5ddb2c5bfec8a025674aba6ed84a2db26277/src/com/ozragwort/code/baekjoon/q14247.java)

---

### 함수형 인터페이스

abstract method가 하나인 인터페이스

```java
public interface Generator {
    Lotto generate();
}
```

요즘 우테코에서 핫한 전략 패턴처럼 함수 객체를 사용하기 위해 익명 클래스를 사용하는 예시이다.

함수형 인터페이스라고 부르는 인터페이스의 인스턴스를 정의할 때, 어딘가에 클래스를 만들고 사용하는게 아니라 저곳에서 바로 클래스를 만들어 사용한다.

정렬하는 방법은 오름차순, 내림차순 외에도 다른 여러 방법이 있을 것인데 어느 클래스를 만들어 쓰지않고 그 곳에 바로 구현해서 사용하는 것이 익명 클래스.

저걸 인터페이스로 만들고 구현체를 만들어 사용하면 그게 바로 전략 패턴인듯하다.(오름차순으로 정렬하는 구현체, 내림차순으로 정렬하는 구현체 등을 만들어 쓰면)

자바 8버전이 되면서 이 인스턴스르 만들 때 람다를 쓸 수 있게 해주었다.

### 람다를 사용하는 경우

```java
public class Foo {
    public void foo() {
        Arrays.sort(length, (o1, o2) -> o1[1] - o2[1]);
    }
}
```

람다와 익명 클래스가 다른 동작을 하는건 아니지만 더 간단하게 구현할 수 있다.

### 실전 사용

[알렉스의 LottoMachineTest - 전략 패턴 인터페이스를 람다로 재정의](https://github.com/yxxnghwan/java-lotto/blob/step2/src/test/java/domain/LottoMachineTest.java)

NumberGenerateStrategy를 사용할 때 람다, 익명 클래스를 사용하여 값을 재정의하는 방법

```java
public interface NumberGenerateStrategy {
    Set<Integer> generateNumbers();
}

// 람다 사용
public class LottoMacineTest {
    private final NumberGenerateStrategy numberGenerateStrategy = () -> Set.of(1, 2, 3, 4, 5, 6);
}

// 익명 클래스 사용
public class LottoMacineTest {
    private final NumberGenerateStrategy numberGenerateStrategy = new NumberGenerateStrategy() {
        @Override
        public Set<Integer> generateNumbers() {
            return Set.of(1, 2, 3, 4, 5, 6);
        }
    };
}
```

두 방법 같은 기능을 하는 numberGenerateStrategy가 만들어 진다. 하지만 람다를 사용하는게 더 간단하고 이해하기 쉽다. 가독성 짱

### 타입 추론에 대한 이야기

보통 람다식에서는 매개변수와 반환값의 타입을 적지 않는다. 컴파일러가 문맥을 살펴 타입을 추론해주기 때문이다.

상황에 따라 컴파일러가 타입을 결정하지 못하기도 한다. 그런 경우 프로그래머가 직접 명시해야한다.

하지만 늘 타입을 적기 힘들고 설계를 잘했으면 타입안적어도 된다. 그러니 에러가 발생하거나 타입을 명시했을 때 코드가 더 명확해지는게 아니라면 타입을 생략하자.

#### 문제가 발생하는 경우

제네릭을 사용하는 경우 람다와 함께 쓸 때는 매우 중요하다.

예를들어 로타입을 사용하면 람다로 함수 객체를 만들때 꼭 타입을 명시해야한다.

```java
public class Foo {
    public void foo() {
        List<Integer> numbers1 = Arrays.asList(1, 2, 3, 4);
        Collections.sort(numbers1, (o1, o2) -> o1 - o2); // 문제 없음
        Collections.sort(numbers1, Comparator.comparingInt(o -> o)); // 비교자 생성 메서드를 사용해도 간단하게 사용 가능
        Collections.sort(numbers1, Comparator.comparingInt(Integer::intValue)); // 이런 식으로도 사용 가능

        List numbers2 = Arrays.asList(1, 2, 3, 4); // 로타입
        Collections.sort(numbers2, (o1, o2) -> o1 - o2); // 에러 발생
        Collections.sort(numbers2, (Integer o1, Integer o2) -> o1 - o2);
        Collections.sort(numbers2, (o1, o2) -> (Integer) o1 - (Integer) o2);
    }
}
```

## 열거형 클래스에서 람다 사용

[이번 미션](https://github.com/woowacourse/java-lotto)에서 사용한 Enum으로 예를 들어본다.

[소스코드](https://github.com/OzRagwort/java-lotto/blob/step2/src/main/java/lotto/domain/LottoPrize.java)

다음은 내가 구현한 코드이다. `match()`에서 맞춘 번호의 갯수, 보너스번호 확인을 하고 등수를 반환한다.

```java
public enum LottoPrize {
    MISS(0, new Money(0)),
    FIFTH(3, new Money(5000)),
    FOURTH(4, new Money(50000)),
    THIRD(5, new Money(1500000)),
    TWICE(5, new Money(30000000)),
    FIRST(6, new Money(2000000000));

    private final int lottoNumberMatchCount;
    private final Money reward;

    LottoPrize(int lottoNumberMatchCount, Money reward) {
        this.lottoNumberMatchCount = lottoNumberMatchCount;
        this.reward = reward;
    }

    public static LottoPrize match(int lottoNumberMatchCount, boolean bonusNumberMatch) {
        if (isTwice(lottoNumberMatchCount, bonusNumberMatch)) {
            return TWICE;
        }

        return Arrays.stream(LottoPrize.values())
                .filter(prize -> prize.lottoNumberMatchCount == lottoNumberMatchCount)
                .filter(prize -> !prize.equals(TWICE))
                .findFirst()
                .orElse(MISS);
    }

    private static boolean isTwice(int lottoNumberMatchCount, boolean bonusNumberMatch) {
        return lottoNumberMatchCount == TWICE.lottoNumberMatchCount && bonusNumberMatch;
    }
}

```

여기서 등수마다 다른 조건을 복잡하게 검사하지 않고 람다를 이용하여 편하게 검사할 수 있다.

각각의 멤버들에게 자신의 당첨 조건을 람다로 적어 가지고 있는다.

나는 BiPredicate를 사용했는지 이외에도 여러 함수 인터페이스가 있다.(44장에서 설명 예정)

```java
public enum LottoPrize {
    MISS(0, new Money(0), (count, bonus) -> count < 3),
    FIFTH(3, new Money(5000), (count, bonus) -> count == 3),
    FOURTH(4, new Money(50000), (count, bonus) -> count == 4),
    THIRD(5, new Money(1500000), (count, bonus) -> count == 5 && !bonus),
    TWICE(5, new Money(30000000), (count, bonus) -> count == 5 && bonus),
    FIRST(6, new Money(2000000000), (count, bonus) -> count == 6);

    private final int lottoNumberMatchCount;
    private final Money reward;
    private final BiPredicate<Integer, Boolean> match;

    LottoPrize(int lottoNumberMatchCount, Money reward, BiPredicate<Integer, Boolean> match) {
        this.lottoNumberMatchCount = lottoNumberMatchCount;
        this.reward = reward;
        this.match = match;
    }

    public static LottoPrize match(int lottoNumberMatchCount, boolean bonusNumberMatch) {
        return Arrays.stream(LottoPrize.values())
                .filter(prize -> prize.match.test(lottoNumberMatchCount, bonusNumberMatch))
                .findFirst()
                .orElse(MISS);
    }
}
```

`match()` 함수가 많이 간단해졌다. 이전에는 early return도 하고 filter도 두개를 사용하였지만 간단하게 람다식만 검사하면 된다.

## 람다 사용 시 주의사항

**람다식은 제목이 없다.**

따라서 한눈에 동작을 이해할 수 있을만큼 간단한것이 좋다.

(책에서는) 길어야 3줄 안에 끝내는것이 좋다.

가독성을 향상시키려다가 오히려 어려워질 수 있다.

## 람다가 짱인가?

람다가 좋은건 맞지만 아직 익명 클래스를 사용해야하는 곳이 있다.

### 1. 여기서 예시로 든 함수형 인터페이스에서는 사용하기 좋지만 추상 메서드가 여러개라면 람다를 쓸 수 없다.

```java
public interface VendingMachine {
    void coke();

    void sprite();
}

public class Foo {
    public void foo() {
        VendingMachine vendingMachine = new VendingMachine() {
            @Override
            public void coke() {
                System.out.println("콜라");
            }

            @Override
            public void sprite() {
                System.out.println("사이다");
            }
        };
        VendingMachine vendingMachine = () -> ???; // 이건 안됨.
    }
}
```

### 2. 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없다.

```java
abstract class Hello {
    abstract void hi();
}

public class Foo {
    public void foo() {
        Hello hello1 = new Hello() {
            @Override
            void hi() {
                System.out.println("하이");
            }
        };
        Hello hello2 = () -> System.out.println("하이"); // 이건 안됨.
        hello2.hi();
    }
}
```

### 3. 람다에서 `this`는 바깥 인스턴스를 가리키고 익명 클래스의 `this`는 자신을 가리킨다. 그래서 함수 객체가 자신을 참조해야하는 코드를 작성하려면 익명 클래스를 써야한다.

```java
interface Coffee {
    void latte();

    default void printPrice() {
        System.out.println(1000);
    }
}

public class Foo {
    public void foo() {
        Coffee coffee1 = () -> this.printPrice(); // 2000

        Coffee coffee2 = new Coffee() {
            @Override
            public void latte() {
                this.printPrice(); // 1000
            }
        };
    }

    public void printPrice() {
        System.out.println(2000);
    }
}
```

### 4. 직렬화를 해야한다면 람다, 익명 클래스 둘 다 쓰지마라.

[직렬화 자료](https://javarevisited.blogspot.com/2011/04/top-10-java-serialization-interview.html#axzz7Mr2mkfPn)

직렬화에 대해 간단하게..

자바 객체 --직렬화--> 다른 데이터 형식 --역직렬화--> 자바 객체

자바 객체를 file, DB, json, xml 등등 다른 형식으로 변경하는것을 직렬화라고 한다.

이런 직렬화 작업은 구현하는 형식, 운영체제 등에 따라 달라지기 때문에 람다보다는 private 정적 중첩 클래스의 인스턴스를 사용하자고 한다.

private 정적 중첩 클래스는 아이템 24에서 설명중이라고 함!

```java
public class FruitCollection {
    private final FruitComparator comparator;
        // ....

    public FruitCollection(Comparator<Fruit> comparator) {
        this.comparator = comparator;
    }

    static class FruitComparator implements Comparator<Fruit>, Serializable {
        private static final long serialVersionUID = 1;

        @Override
        public int compare(Fruit o1, Fruit o2) {...}
    }
}
```
[정적 중첩 클래스 예시 by 자바봄](https://javabom.tistory.com/66)
