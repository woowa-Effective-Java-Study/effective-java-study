# 아이템 7 - 다 쓴 객체 참조 해제

## 메모리 누수
### 메모리 누수가 발생하는 Stack 코드 예제
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size]; // elemenets가 다 쓴 참조를 여전히 가지고 있게 된다.
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```
- 다 쓴 참조 : 앞으로 다시 쓰지 않을 활성 영역 밖의 참조

![image](https://user-images.githubusercontent.com/76840965/159342492-62a9c6bd-f45b-415a-9502-eb377dd97f35.png)
![image](https://user-images.githubusercontent.com/76840965/159342539-2d313580-6815-429f-87be-40b88714e214.png)

- 가비지 컬렉터는 어떤 객체에 대한 참조가 하나라도 살아있으면 해당 객체뿐 아니라 그 객체가 참조하는 모든 객체들을 회수해가지 못한다. -> 메모리 누수 발생

### 해결 방법
- 해당 참조를 다 썼을 때 null 처리(참조 해제)를 해준다.
```java
public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
```
<br>

## 객체 참조의 null 처리
- 잦은 null 처리는 프로그램을 필요 이상으로 지저분하게 만듦 -> 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.

### 객체 참조 null 처리가 필요한 경우
1. 자기 메모리를 직접 관리하는 클래스(위의 예제에서의 Stack 클래스)
2. 캐시
    - 캐시 엔트리의 유효 기간을 정확히 정의하기가 어렵기 때문에 메모리 누수가 발생할 수 있다.
    - 해결 방법
      - 캐시 위부에서 키를 참조하는 동안만 엔트리가 살아있는 캐시가 필요한 경우 : WeakHashMap
      - 주기적으로 쓰지 않는 엔트리 청소 : ScheduledThreadPoolExecutor, LinkedHashMap의 removeEldestEntry
3. 리스너(콜백)
    - 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는 경우 콜백이 계속 쌓여서 문제가 발생한다.
    - 해결 방법
      - 콜백을 약한 참조로 저장 : WeakHashMap

### WeakHashMap
```java
import java.util.Map;
import java.util.WeakHashMap;

public class WeakHashMapTest {

    public static void main(String[] args) {
        Map<Integer, Stack> weakMap = new WeakHashMap<>();

        Stack s1 = new Stack();
        Stack s2 = new Stack();
        weakMap.put(20, s1);
        weakMap.put(450, s2);

        // 정상적으로 2개 출력
        for (Map.Entry<Integer, Stack> elem : weakMap.entrySet()) {
            System.out.printf("%d: %s\n", elem.getKey(), elem.getValue());
        }

        // 강제 Garbage Collection
        s2 = null;
        System.gc();

        // WeakHashMap 에서 450 키가 자동으로 사라진 것을 확인할 수 있음
        for (Map.Entry<Integer, Stack> elem : weakMap.entrySet()) {
            System.out.printf("%d: %s\n", elem.getKey(), elem.getValue());
        }
    }
}
```
