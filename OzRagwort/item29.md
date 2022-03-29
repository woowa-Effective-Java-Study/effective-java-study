# item 29. 이왕이면 제네릭 타입으로 만들라

## item 7(다 쓴 객체 참조를 해제하라)에서 나온 Stack 클래스의 문제점

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.EmptyStackException;

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
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 이 코드가 없으면 다 쓴 참조를 여전히 가지고 있게 된다.
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size);
    }
}
```

![image](https://user-images.githubusercontent.com/76840965/159342492-62a9c6bd-f45b-415a-9502-eb377dd97f35.png)
![image](https://user-images.githubusercontent.com/76840965/159342539-2d313580-6815-429f-87be-40b88714e214.png)

하지만 이 클래스는 Object로 리턴이 되기 때문에 `pop()`을 하고나서 사용하기 위해 형변환을 해야한다.

Stack 클래스에서 제네릭을 사용하면 형변환을 하지 않고 바로 사용할 수 있게 만들 수 있다.

## 제네릭으로 Stack 구현하기

1. Object[]에 바로 제네릭 적용하기
2. 내부에서는 Object[]를 사용하고 push(), pop()할 때 형변환해서 반환하기

### Object[]에 바로 제네릭 적용하기

```java
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size);
    }
}
```

Object를 E로 수정하여 제네릭을 적용한 코드다.

여기서는 하나의 자료형을 나타내기 때문에 Element의 E를 사용하는듯.

하지만 이 코드는 에러가 발생한다.
이유는 [배열과 제네릭의 차이점](https://github.com/woowa-Effective-Java-Study/effective-java-study/blob/main/OzRagwort/item28.md#2%EB%B0%B0%EC%97%B4%EC%9D%80-%EC%8B%A4%EC%B2%B4%ED%99%94reify%EA%B0%80-%EB%90%9C%EB%8B%A4-%EC%A0%9C%EB%84%A4%EB%A6%AD%EC%9D%80-%EC%BB%B4%ED%8C%8C%EC%9D%BC%EC%9D%B4-%EB%90%9C-%ED%9B%84-%EC%86%8C%EA%B1%B0%EB%90%9C%EB%8B%A4)
에서 복습하자.

이해가 되지
않는다면 [배열보다는 리스트를 사용하라](https://github.com/woowa-Effective-Java-Study/effective-java-study/blob/main/OzRagwort/item28.md#%EB%AC%B8%EC%A0%9C%EA%B0%80-%EB%90%98%EB%8A%94-%EC%BD%94%EB%93%9C%EC%9D%98-%EC%9C%84%EC%B9%98%EB%8A%94-%EC%96%B4%EB%94%94%EC%9D%BC%EA%B9%8C)
도 다시 보자!

그래도 이 방법으로 구현해야 한다면 제네릭으로 배열을 생성하지 못하도록 하는 제약을 우회해야한다.

Object로 배열을 만들고 형변환하는 것이다.

하지만 컴파일러에서 오류 대신 비검사 경고가 발생한다. 그리고 데이터가 무엇이 오는지에 따라 오류가 발생할 수 있다.

내가 보기에 절대 문제가 발생하지 않고 이 클래스는 영원히 수정되지 않을 것이라고 한다면 `@SuppressWarnings("unchecked")`로 경고를 숨길 수도 있다.

```java
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size);
    }
}
```

### 내부에서는 Object[]를 사용하고 push(), pop()할 때 형변환해서 반환하기

두번째 방법은 `push()`와 `pop()` 할 때 제네릭을 적용하고 내부에서는 Object[]를 사용하면서 제네릭을 적용하는 것이다.

이 경우 `pop()` 에서 에러가 발생할 수 있다.

제네릭은 실체화가 되지 않기 때문에 Object배열에 있는 데이터의 형식이 유일한지 확인할 수 없기 때문이다.

하지만 무조건 제네릭 형식의 데이터만이 저장되도록 하면 문제가 없다.

Stack을 예로들면 유일하게 데이터를 추가하는 메서드가 `push()`인데 여기서 제네릭으로 매개변수를 받아 사용하면 문제가 없다.

```java
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack<E> {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();

        @SuppressWarnings("unchecked")
        E result = (E) elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size);
    }
}
```

## 제네릭을 적용 사례 중 우리가 자주 사용하는 ArrayList도 두번째 방식을 사용하고 있다.

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {

    private static final int DEFAULT_CAPACITY = 10;
    private static final Object[] EMPTY_ELEMENTDATA = {};
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    transient Object[] elementData;
    private int size;

    private void add(E e, Object[] elementData, int s) {
        if (s == elementData.length)
            elementData = grow();
        elementData[s] = e;
        size = s + 1;
    }

    public boolean add(E e) {
        modCount++;
        add(e, elementData, size);
        return true;
    }

    public E set(int index, E element) {
        Objects.checkIndex(index, size);
        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

    public E get(int index) {
        Objects.checkIndex(index, size);
        return elementData(index);
    }

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }
}
```

## 그래서 무엇을 써야하는가?

(조슈아가 말하길) 두 방법은 서로 나름대로 장점이 있다고 함.

첫 번째 방법은 가독성이 더 좋다. 코드도 더 짧다. 형변환도 최초에 한번만 해주면 된다.

두 번째 방법은 원소를 읽을 때마다 해줘야한다.

그래서 현업에서는 절대 문제가 없게 코드를 작성하겠다고 마음먹고 첫 번째 방법을 더 선호하기도 한다.

그러나 상황에 따라 힙(heap) 오염을 일으키기도 한다고 함.(item 32)

힙 오염이 마음에 걸리면 두 번째 방식을 사용하기도 함.

적어도 예시 Stack 코드에서는 문제 없다고 함.

# 정리

클라이언트가 직접 형변환하여 사용해야한다면 제네릭을 적용하는게 더 좋다.

새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 해라!

---

## 두 번째 방식에 문제는 없을까?

```java
public class Main {
    public static void main(String[] args) {
        Test<String> test = new Test<>();
        test.add("1"); // 1
        test.add(1); // 2
        System.out.println(test.get(0)); // 3
        System.out.println(test.get(1)); // 4
    }

    public static class Test<E> {
        private Object[] array = new Object[100];
        private int size = 0;

        public void add(Integer i) {
            array[size++] = i; // 5
        }

        public void add(E i) {
            array[size++] = i; // 6
        }

        @SuppressWarnings("unchecked")
        public E get(int index) {
            return (E) array[index]; // 7
        }
    }
}
```

1. 컴파일 에러가 발생하는 곳을 모두 찾으면?
2. 런타임 에러가 발생하는 곳을 모두 찾으면?
3. 해결 방법은 프로그래머가 코드를 잘 작성하고 문서화하는 것 밖에 없는건가?

##  

```java
public class WebApplication {
    public static void main(String[] args) {

        Test test = new Test<String>();
        test.add("1"); // 1
        test.add(1L); // 2
        test.add(1); // 3
        System.out.println(test.get(0)); // 4
        System.out.println(test.get(1)); // 5
        System.out.println(test.get(2)); // 6
    }

    public static class Test<E> {
        private Object[] array = new Object[100];
        private int size = 0;

        public void add(E i) {
            array[size++] = i; // 7
        }

        @SuppressWarnings("unchecked")
        public E get(int index) {
            return (E) array[index]; // 8
        }
    }
}
```

1. 컴파일 에러가 발생하는 곳을 모두 찾으면?
2. 런타임 에러가 발생하는 곳을 모두 찾으면?
