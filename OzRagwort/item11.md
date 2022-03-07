# item 11. equals를 재정의하려거든 hashCode도 재정의하라

---

# 주제를 시작하기 전 문제

아래와 같은 클래스가 있을 때

```java
public class Money {
    private final int money;
}

public class LottoNumber {
    private static final Map<Integer, LottoNumber> cacheLottoNumbers = new HashMap<>();

    public static LottoNumber valueOf(int number) {
        validateRange(number);
        if (!cacheLottoNumbers.containsKey(number)) {
            cacheLottoNumbers.put(number, new LottoNumber(number));
        }
        return cacheLottoNumbers.get(number);
    }

    @Override
    public int hashCode() {
        return Objects.hash(number);
    }
}
```

1 ~ 10까지 결과값을 생각해보자.

```java
public class Foo {
    public void foo() {
        Money m1 = new Money(1);
        Money m2 = new Money(1);
        Money m3 = new Money(2);
        LottoNumber ln1 = LottoNumber.valueOf(1);
        LottoNumber ln2 = LottoNumber.valueOf(1);
        LottoNumber ln3 = LottoNumber.valueOf(2);
        Integer i1 = Integer.valueOf(1);
        Integer i2 = new Integer(1);
        Integer i3 = new Integer(2);
        Integer i4 = new Integer(2);

        HashSet<Money> moneySet = new HashSet<>(List.of(m1, m2, m3));
        HashSet<LottoNumber> lottoNumberSet = new HashSet<>(List.of(ln1, ln2, ln3));
        HashSet<Integer> integerSet = new HashSet<>(List.of(i1, i2, i3, i4));

        System.out.println(m1.hashCode() == m2.hashCode()); // 1
        System.out.println(m1.hashCode() == m3.hashCode()); // 2
        System.out.println(ln1.hashCode() == ln2.hashCode()); // 3
        System.out.println(ln1.hashCode() == ln3.hashCode()); // 4
        System.out.println(i1.hashCode() == i2.hashCode()); // 5
        System.out.println(i1.hashCode() == i3.hashCode()); // 6
        System.out.println(i3.hashCode() == i4.hashCode()); // 7
        System.out.println(moneySet.size()); // 8
        System.out.println(lottoNumberSet.size()); // 9
        System.out.println(integerSet.size()); // 10
    }
}
```

---

# <핵심만 쏙쏙 버전>

## 초간단 핵심 요약 정리

- (equals가 재정의되었다고 가정)
- HashMap의 키, HashSet의 값 등은 중복을 허용하지 않는데 이 때 같음을 판단하는 기준이 hashCode와 equals다.
- 간단하게 말하면 hashCode가 같고 equals가 참이면 HashXXX에서 중복으로 판단하고 값이 추가되지 않는다.
- 그런데 HashXXX에서 값을 찾을 때(`contains()`)도 hashCode와 equals로 찾는다.
- hashCode가 같으면 equals로 비교한다.

```java
public class HashMap<K, V> extends AbstractMap<K, V> {
    // ...
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        // ... 
        // hashCode와 equals가 같으면 중복이라 해시테이블에 추가하지 않음
    }

    // ...
    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }

    final Node<K, V> getNode(int hash, Object key) {
        // ... 
        // hashCode가 같은 녀석들 중에서 equals가 같은거 찾아서 있으면 반환 없으면 null
    }
}
```

문제는 equals가 같은 동등한 객체여도 hashCode를 재정의 하지 않거나 다른 값이 나오도록 재정의하면 해시 테이블에 저장이 된다.

그렇기 때문에 equals가 같으면 hashCode값이 같아야 한다! 그래서 equals를 재정의할꺼면 hashCode도 재정의 하라는 것!

hashCode값이 같아도 equals는 다른 수도 있다. 하지만 hashCode값의 충돌이 많으면 4번째 과정에서 성능이 안좋아지므로 충돌은 가능하면 안되도록 hashCode를 재정의하자.

# <주절주절 버전>

우리가 참 많이 쓰는 메서드 중 equals라는게 있다. Object.equals의 명세는 다음과 같다.

- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 값은 항상 같은 값을 반환해야 한다. 단, 재 실행 시 이 값이 달라질 수 있다.
- equals가 두 객체를 같다고 판단했다면 두 객체의 hashCode 값은 같다.
- equals가 두 객체를 다르다고 판단해도 hashCode는 같은 수도 있다. 다만 다른 값이어야 해시태이블의 성능이 좋아진다.

item 11과 연관된 명세는 두번째이다. 즉 equals가 같다면 hashCode값이 같아야 한다.

예시)

```java
public class Money {
    private final int money;
}

public class LottoNumber {
    private static final Map<Integer, LottoNumber> cacheLottoNumbers = new HashMap<>();

    public static LottoNumber valueOf(int number) {
        validateRange(number);
        if (!cacheLottoNumbers.containsKey(number)) {
            cacheLottoNumbers.put(number, new LottoNumber(number));
        }
        return cacheLottoNumbers.get(number);
    }

    @Override
    public int hashCode() {
        return Objects.hash(number);
    }
}
```

```java
public class Foo {
    public void foo() {
        Map<Money, Integer> moneyMap = new HashMap<>();
        moneyMap.put(new Money(1000), 1);
        moneyMap.get(new Money(1000)); // 1

        Map<LottoNumber, Integer> numberMap = new HashMap<>();
        numberMap.put(new LottoNumber(1), 1);
        numberMap.get(new LottoNumber(1)); // 2
    }
}
```

1과 2의 값은 무엇일까? 1은 null이 2는 1이 반환된다. 그 이유는 Money는 hashCode()가 재정의 되어있지 않고 LottoNumber는 재정의가 되어있기
때문이다. `new Money(1000).equals(new Money(1000))` 는 false 이고 `LottoNumber.valueOf(1).equals(LottoNumber.valueOf(1))`는 true
이기 때문이다.

## 그럼 어떻게 hashCode를 작성해야할까?

고려할 점!! → 최대한 32비트 정수 범위에서 균일하게 분배해야한다.

질문) 왜 32비트인가?

hashCode 작성 요령 예시)

```java
public class Foo {
    // Integer
    public static int hashCode(int value) {
        return value;
    }

    // Long
    public static int hashCode(long value) {
        return (int) (value ^ (value >>> 32));
    }

    // String
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            hash = h = isLatin1() ? StringLatin1.hashCode(value)
                    : StringUTF16.hashCode(value);
        }
        return h;
    }

    // Objects.hash().hashCode()
    public static int hashCode(Object a[]) {
        if (a == null)
            return 0;

        int result = 1;

        for (Object element : a)
            result = 31 * result + (element == null ? 0 : element.hashCode());

        return result;
    }
}
```

1. int 변수 result를 선언 후 c로 초기화 한다. c는 해당 객체의 첫 핵심 필드(equals로 비교하는 필드)의 해시코드.
2. 핵심 필드들을 각각 다음 작업을 한다.
    1. 원시값이면 해당 박싱 클래스 Type.hashCode(값)을 한다.(int : Integer)
        1. 원시값이 아니면 알아서 hash값을 가져오되 null이면 0을 가져온다.
        2. 배열이라면 모든 값을 1)부터 계산하면된다. 이미 만들어진 Arrays.hashCode 쓰면된다.
    2. 2.a에 작업한 해시코드 값으로 result를 다음 식으로 갱신한다.

   `result = 31 * result + c`

3. 다 끝나면 result를 반환한다. 이게 우리가 보는 해시코드값이다.

```java
public class Test {
    @Test
    public void test() {
        Money money = new Money(1);
        LottoNumber lottoNumber = LottoNumber.valueOf(1);

        assertThat(money.hashCode() == lottoNumber.hashCode()).isTrue();
    }
}
```

hashCode 메서드가 위 방식대로 되어있다면(Objects.hash()을 쓰면) 두 해시값은 타입이 다르지만 32로 같다.

## 해시코드 계산에 가장 중요한 것

**여기서 핵심은 equals로 비교하는 모든 필드는 꼭 해시코드 계산에 들어가야하고 equals로 비교하지 않는 값은 꼭 해시코드 계산에 빠져야한다.**

# 왜 해시코드 값이 고정이면 안될까?

```java
public class BadCase {
    @Override
    public int hashCode() {
        return 1;
    }
}
```

위 방식으로 해시코드를 정의하면 모든 객체의 해시코드 값이 같다.

해시 테이블에는 해시값이 같으면 데이터는 LinkedList 형식으로 데이터가 쌓여간다.

그럼 해시 테이블에 저장될 때 해시코드 값이 모두 같으면 모두 하나의 LinkedList에 포함된다.

최적은 모든 해시값이 다를경우 탐색 시간이 O(1)이 되고 최악은 모든 해시값이 같아 탐색 시간이 O(N)이 되는것이다. 

하지만 충돌이 생기는 값이 8개를 초과하면 Tree로 저장이 되어 O(log(N))이 된다.

참고자료 : [Java HashMap은 어떻게 동작하는가?](https://d2.naver.com/helloworld/831311)

그래서 해시 코드값이 달라야 성능에 좋다는 것이다.

## TMI) 왜 31일까?

31은 소수이기 때문이다.

계속 곱하며 더해주는데 소인수분해가 잘 되는 값을 사용하면 중복(충돌)이 자주 발생한다. 이것을 막기 위해 소수를 이용한다.

왜 같은 소수인데 2, 3도 아니고 14292471727423은 아닐까?

수가 너무 작으면 충돌이 자주 발생하고 너무 크면 연산량도 많고 자료형의 오버플로우가 발생할 수 있다.
