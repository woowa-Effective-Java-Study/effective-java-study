# item 27. 비검사 경고를 제거하라

## 비검사 경고는 할 수 있는 한 모두 제거해야 한다.

### 비검사 경고

영어로 unchecked warning이다. 컴파일러 경고 중 하나이다.

javac 명령줄에 인수로 -Xlint:uncheck 옵션을 추가하면 어디가 어떻게 잘못되었는지 알려준다.

```java
public class Foo {
    public void foo() {
        Set<Car> cars = new HashSet();   // 문제 발생 코드
        Set<Car> cars = new HashSet<>(); // 문제 해결 코드
    }
}
```

```text
Venery.java:4: warning: [unchecked] unchecked conversion
                Set<Car> cars = new HashSet();
                                ^
  required: Set<Car>
  found:    HashSet
```

에러가 아닌 경고이기 때문에 실행은 된다.

제네릭을 사용하면서 로타입을 쓰거나 잘못 설정하면 보통 발생한다.

### 이런 비검사 경고를 모두 제거해야한다.

비검사 경고를 모두 제거한다면 그 콛는 타입 안전성을 보장할 수 있다.

즉 런타임중 `ClassCastException`이 발생할 일이 없다.

## 경고를 제거할 수 없지만 안전하다고 확신하는 경우 경고를 무시하는 방법

`@SuppressWarnings("unchecked")` 애너테이션을 달아주면 경고를 무시할 수 있다.

하지만 타입 안전성을 검증하지 않기 때문에 `ClassCastException`이 발생할 수 있다.

`@SuppressWarnings`의 옵션 정리

1. all : 모든 경고를 억제
2. cast : 캐스트 연산자 관련 경고 억제
3. dep-ann : 사용하지 말아야 할 주석 관련 경고 억제
4. deprecation : 사용하지 말아야 할 메소드 관련 경고 억제
5. fallthrough : switch문에서의 break 누락 관련 경고 억제
6. finally : 반환하지 않는 finally 블럭 관련 경고 억제
7. null : null 분석 관련 경고 억제
8. rawtypes : 제네릭을 사용하는 클래스 매개 변수가 불특정일 때의 경고 억제
9. unchecked : 검증되지 않은 연산자 관련 경고 억제
10. unused : 사용하지 않는 코드 관련 경고 억제

출처 : [@SuppressWarnings 이건 뭐지~?](https://jinwoonote.tistory.com/entry/SuppressWarnings-%EC%9D%B4%EA%B1%B4-%EB%AD%90%EC%A7%80)

## `@SuppressWarnings`은 어디서 사용할 수 있나

`@SuppressWarnings`는 선언하는 단계에서만 달 수 있다.

`@SuppressWarnings`는 지역변수 선언부터 클래스 전체까지 어디든 선언할 수 있다.

하지만 의도하지 않은 곳에서 발생하는 경고를 놓칠 수 있으니 가능한 좁은 범위에 적용하자.

- ArrayList의 toArray() 메서드

```java
import java.util.ArrayList;

public class ArrayList {
    // 실제 코드
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // @SuppressWarnings("unchecked") // 가능하면 여기있는게 좋기는 함
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
}
```

`@SuppressWarnings`를 사용할 때는 왜 경고를 무시해도 되는지 주석으로 남겨야 한다.

하지만 비검사 경고는 언제든 `ClassCastException`이 발생할 수 있으니 최선을 다해 제거하자!

---

## TMI) List를 배열로 바꿀 때 왜 원시값 배열은 만들 수 없을까?

```java
import java.util.ArrayList;
import java.util.List;

public class Foo {
    public void foo() {
        List<String> stringList = List.of("1", "2", "3");

        stringList.stream().map(Integer::parseInt).toArray(Integer[0]); // 이건 된다.
        stringList.stream().map(Integer::parseInt).toArray( int[0]);     // 이건 안된다.
    }
}
```

정답은 ArryaList.toArray(T[] a)의 코드를 확인보며 고민해보자!
