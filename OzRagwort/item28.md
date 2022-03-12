# item 28. 배열보다는 리스트를 사용하라

## 배열과 제네릭 타입의 차이점

### 1. 배열은 공변이고 제네릭은 불공변이다.

공변은 간단하게 말해서 String[]은 Object[]의 하위 타입이 되고 컴파일도 된다는 말이다. 하지만 로타입에서 잘못된 타입의 데이터를 추가할 때 처럼 이곳에서도 잘못된 타입을 넣으면 에러가 발생한다.

하지만 제네릭 타입은 불공변이다. List<Object>와 List<String>은 아예 다른 타입이 된다. 상/하위 타입이 아니기 때문에 컴파일부터 에러가 발생한다.

```java
import java.util.ArrayList;

public class Foo {
    public void foo() {
        Object[] array = new Long[1];
        array[0] = 1;
        array[0] = "1"; // 실행 시 런타임에서 에러(ArrayStoreException)

        List<Object> list = new ArrayList<Long>(); // 컴파일 과정에서 에러
        list.add(1);
        list.add("1");
    }
}
```

### 2.배열은 실체화(reify)가 된다. 제네릭은 컴파일이 된 후 소거된다.

배열은 런타임 중 자신이 담으려는 원소의 타입을 인지하고 확인한다.

그래서 Object[]을 Long[]으로 선언하면 타입은 Object[]이지만 원소가 Long인지 아닌지 계속 확인하게된다.

하지만 제네릭 타입은 컴파일이 되면 타입이 "소거"된다.

소거가 되면 타입 정보가 사라지고 로타입이 되어 런타임 중 타입을 알 수가 없다.

그래서 이런 제네릭 타입을 실체화 불가 타입이라고도 한다.

소거 방식을 사용하는 이유는 제네릭 도입 전의 코드와의 호환성을 위함이다.

하지만 List<?>, Map<?,?>과 같은 비한정적 와일드카드 타입은 소거되지 않는다고 한다.

## 그래서 제네릭 타입은 배열로 만들 수 없다.

- new List<E>[]
- new List<String>[]
- new E[]

등 모두 사용이 불가능하다.

그 이유는 제네릭은 컴파일 시 `ClassCastException`를 발생시키고 런타임중에는 검사하지 않는다고 설계했기 때문이다.(소거했기 때문)

### 문제가 되는 코드의 위치는 어디일까?

```java
import java.util.List;

public class Foo {
    public void foo() {
        List<String>[] listArray = new List<String>[1]; // 1
        List<Integer> list = List.of(30);               // 2
        Object[] objects = listArray;                   // 3
        objects[0] = list;                              // 4
        String s = listArray[0].get(0);                 // 5
    }
}
```

.

.

.

실제 에러가 발생하는 위치는 1번이고 문제가 발생할 수 있는 위치는 5번이다.

자바의 문법상 2,3,4,5에는 문제가 되는 부분이 없다. 런타임중에는 List<String>와 List<Integer>가 모두 List가 되기 때문이다.

위 코드는 런타임중에는 소거가 되면서 아래 코드와 같이 동작한다.

```java
import java.util.ArrayList;
import java.util.List;

public class Foo {
    public void foo() {
        List[] listArray = new List[1];
        List list = List.of(30);
        Object[] objects = listArray;
        objects[0] = list;
        String s = listArray[0].get(0);
    }
}
```

---

## 이런 문제를 해결하기 위해서는 배열대신 리스트를 써라

E[]는 List<E>로 사용하면 된다.

둘 다 여러 원소를 가지는 자료구조이기 때문이다.

보통 리스트보다 배열이 성능이 더 좋다고 하지만 대신 타입 안전성과 상호운용성 등이 좋아진다.

타입 안전성이 좋은 이유는 컴파일 에러로 타입에러를 잡아주고 혹시라도 잘못된 부분은 비검사 경고로 알려주기 때문이다.

런타임 중 절대 `ClassCastException`이 발생하지 않는 것은 우리 마음에 편안함을 가져다 준다.(네오톤)

상호운용성이 좋다는 말은 다른 언어와의 호환성이 좋다는 것일수도 있고 다른 타입으로의 변경이 쉽다는 말일수도 있는것 같다.

예를들면 Kotlin과 Java는 상호 운용성을 염두하고 설계된 언어이므로 서로의 코드를 서로의 언어에서 호출할 수 있다고 한다.

다른 타입으로의 변경이 쉽다는것은 배열을 List로 바꾸려면 Arrays.asList()바꿔야 하지만 List는 toArray()로 객체에서 바로 바꿀 수 있기 때문이 아닐까..?

---

## 배열과 리스트에 대해서

String[]와 List<String>를 비교해보자.

#### ArrayList<String>()의 경우

ArrayList는 데이터를 저장할 때 Object[]을 사용한다.

따라서 탐색하는 속도는 배열과 큰 차이가 없다.

그리고 List는 size라는 인스턴스 변수를 가지고 있는데 list.size() 에서 반환되는 값이 이 size 변수의 값이다.

add()는 매번 size를 1씩 더해주고 그 Object[size]에 값을 할당하는 메서드다.

이때 Object[]의 length가 size보다 작다면 기존 Object[]보다 1사이즈 더 큰 배열을 new로 만들어 할당한다.

remove()는 Object[]에서 삭제하는 index 뒤의 데이터를 한 index씩 앞으로 저장하고 맨뒤 데이터를 null로 바꾼다. 그리고 size를 1뺀다.

contains를 보면 모든 배열을 탐색하면서 equals로 비교하여 포함 여부를 판단한다.(그래서 equals 재정의를 하지 않으면 contains가 제대로 동작하지 않음)

주관적인 ArrayList의 장점 정리

- item의 주제처럼 타입 안전성을 보장한다.
- 배열을 사용하기 때문에 배열이 할 수 있는건 다 할 수 있다.
- 배열과 다르게 정적으로 쓸 수 있다.(사용자 입장에서)
- 매번 size를 늘리면서 new로 새로운 배열을 만들기 싫다면 처음부터 배열의 크기를 정할 수 있음(new ArrayList<>(initialCapacity))
- stream()을 바로 사용 가능하다.(배열은 제네릭 타입으로 바꾸고 사용해야 함)

**정리**하자면 ArrayList를 두고 배열을 쓸 이유가 없는 것 같다.

다른 XXXList는 이렇고 저렇고 그래서 배열써야합니다! 라고 한다면 -> 결국 배열 사용할바에 ArrayList쓰세요. 결국 List 쓰세요.

```java
public class Foo {
    public void foo() {
        List<String> a = new ArrayList<>();
        List<String> b = new ArrayList<>(10);
        String[] c = new String[10];

        a.stream().count();
        Arrays.stream(c).count();
    }
}
```
