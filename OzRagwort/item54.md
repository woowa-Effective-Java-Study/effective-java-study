# item 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

## 따라하면 안되는 코드

```java
public class CheeseStore {
    private final List<Cheese> cheesesInStock = new ArrayList<>();

    // 치즈 재고가 남아 있다면 반환하고 없으면 null을 반환.
    public List<Cheese> getCheese() {
        return cheesesInStock.isEmpty()
                ? null
                : new ArrayList<>(cheesesInStock);
    }

    // 스틸턴 치즈가 있는지 확인하는 메서드
    public boolean hasStilton() {
        List<Cheese> cheeses = getCheese();
        // null인지 한번 확인을 해야한다!
        if (cheeses != null && cheeses.contains(Cheese.STILTON)) {
            return true;
        }
        return false;
    }
}
```

비었다는 의미를 가진 반환을 할 때 null을 사용한다면 위 코드(`cheeses != null`)처럼 NullPointerException을 방어하는 코드를 추가해야한다.

## null을 반환하는게 더 좋을 수도 있을까?

null을 반환하면 빈 값을 반환하기위해 새로운 객체를 만들지 않아도 된다.(`new ArrayList<>()` 등)

하지만 이것은 두 가지 면에서 틀렸다.

1. 빈 값을 나타내는 새로운 객체를 만드는게 성능에 크게 영향을 주지 않는다.
2. 빈 컬렉션이나 배열은 새로 할당하지 않고도 반환할 수 있다.(캐싱이 되어있음)

결국은 빈 컬렉션을 반환할 때 null을 쓰지 않고 빈 컬렉션을 보내자!

```java
public class CheeseStore {
    private final List<Cheese> cheesesInStock = new ArrayList<>();

    public List<Cheese> getCheese() {
        return new ArrayList<>(cheesesInStock);
    }
}
```

이렇게 쓰자! 비었으면 빈대로 있으면 있는대로~

## 빈 불변 컬렉션

자바에는 빈 컬렉션을 이미 만들어서 캐싱하고 있다.

- Collections.emptyList()
- Collections.emptyMap()
- Collections.emptySet()

빈 불변 컬렉션을 쓰면 이런식으로도 가능하다!

```java
public class CheeseStore {
    private final List<Cheese> cheesesInStock = new ArrayList<>();

    public List<Cheese> getCheese() {
        return cheesesInStock.isEmpty()
                ? Collections.emptyList()
                : new ArrayList<>(cheesesInStock);
    }
}
```

배열도 그렇다. 배열로 반환할 값이 없다고해도 차라리 `new Integer[0]`를 반환하도록 하자!

```java
import java.util.*;

public class CheeseStore {
    private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

    private final List<Cheese> cheesesInStock = new ArrayList<>();

    public Cheese[] getCheese() {
        return cheesesInStock.isEmpty()
                ? EMPTY_CHEESE_ARRAY
                : cheesesInStock.toArray(new Cheese[0]);
//              이건 안좋은 방식(미리 할당을 하면 그만큼 성능, 메모리 손해
//              : cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
    }
}
```

- `cheesesInStock.toArray(new Cheese[0])`

![image](./img/54_1.png)

- `cheesesInStock.toArray(new Cheese[10])`

![image](./img/54_2.png)

[출처 : 간단한것들 구조 보고싶을 때 들어가는 곳](https://pythontutor.com/java.html)

---

# null은 왜 안좋을까?

> null을 처음 설계했다는 토니 호어가 한 말 : The Billion Dollar Mistake(10억 달러짜리 실수)
> 

null을 사용하는 곳에서는 무조건 null 방어용 코드를 작성해야한다.(`if (cheeses != null)`와 같은)

이런 반복문이 쌓이고 쌓이고 사용자가 쌓이고 쌓이면 성능하락과 비용상승으로 이루어질 수 있다.

null을 전혀 사용하지 않는 상태를 보통 `Nullable`, `Null safety` 라고한다.

개발자 입장에서 NullPointerException을 전혀 신경쓰지 않아도 되고 방어적 코드가 없는 만큼 성능하락도 없다.

# 빈 불변 컬렉션은 무조건 좋은가?

![image](./img/54_3.png)

![image](./img/54_3.png)

상황 : 로또 미션에서 금액이 부족해서 로또를 사지 못하는 경우 로또 List를 빈 List로 만들어 출력하는 메서드의 파라미터로 주는 상황

요약을 하자면 앞으로 이 프로그램과 로직이 어떻게 변할지 모르는데 '절대' 사용하지 않는다고 확신할 수 있을까?

혹시라도 나중에 저 List를 사용하면 예외가 발생할 텐데 그냥 `new ArrayList<>()`를 사용하면 지금도 앞으로도 예외가 발생할 부분이 없다.

하지만 빈 불변 컬렉션을 사용하는게 좋은 상황이 있을 수도 있을테니 참고만 하자!
