# item 9. try-finally보다는 try-with-resources를 사용하라

> 꼭 회수해야 하는 자원을 다룰 때는 try-finally대신 try-with-resources를 꼭 사용하자

## 꼭 `close()`해줘야 하는 클래스들

(11버전 기준)

- [java.io 관련](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/package-summary.html)
    - InputStream, OutputStream, FileInputStream 등 Stream 류
    - BufferedReader, BufferedWriter 등 Reader, Writer 류
- [java.sql.Connection](https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Connection.html)

이런 클래스들은 `close()`를 해줘야하는데 클라이언트가 놓치기 쉽고 누적되면 성능 문제가 되기도 한다.

전통적으로는 try-finally가 쓰였다고 한다.

## try-finally 방식으로 close()를 하려면

예시

```java
public class Foo {
    public String foo() {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
}
```

하지만 close해야할 자원이 둘 이상이라면

```java
public class Foo {
    public void foo() {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        try {
            BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
            try {
                bw.write(br.readLine());
                bw.flush();
            } finally {
                bw.close();
            }
        } finally {
            br.close();
        }
    }
}
```

와 같이 사용할 수 있다. 이 코드는 크게 두 가지 문제가 있다.

1. 코드가 복잡하다. 그래서 보기싫고 이해하기 어렵다.
2. close()를 늘 해줘야하고 하지않으면 성능 문제가 발생할 수 있다.
3. 에러 발생 시 에러를 추적하기 어려워 진다.

#### 에러 발생 시 에러를 추적하기 어려워 진다.

```java
public class MyResource implements AutoCloseable {
    public void doSomething() throws FirstError {
        System.out.println("doing something");
        throw new FirstError();
    }

    @Override
    public void close() throws SecondException {
        System.out.println("clean my resource");
        throw new SecondError();
    }
}

public class FirstError extends RuntimeException {
}

public class SecondError extends RuntimeException {
}

public class AppRunner {
    public static void main(String[] args) {
        MyResource myResource = new MyResource();
        try {
            myResource.doSomething();
        } finally {
            myResource.close();
        }
    }
}
```

[코드 출처](https://velog.io/@lychee/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-%EC%95%84%EC%9D%B4%ED%85%9C-9.-try-finally-%EB%8C%80%EC%8B%A0-try-with-resources%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC)

위 코드에서 try문을 실행하면 MyResource의 `doSomething()`을 실행한다.

`doSomething()`은 FirstError()를 발생시킨다.

그 후 에러가 발생되었더라도 finally가 실행되면서 재정의된 `close()`가 실행된다.

`close()`에서는 SecondError()가 발생된다.

이런 경우 우리는 try에서 발생된 FirstError() 예외를 확인할 수 있어야하지만 실제로는 SecondError 예외를 보게된다.

따라서 문제 발생 시 디버깅이 어려워진다.

---

## try-with-resources

try-with-resources는 Java 7에서 추가되었다.

### 선행 조건

우선 try-with-resources를 이용하려면 close()하려는 클래스에 `AutoCloseable`를 반드시 구현해야 한다.

`AutoCloseable`은 close()만을 하도록하는 인터페이스다. 기본적으로 위에서 말한 `java.io`, `java.sql.Connection`등은 `AutoCloseable`를 구현하고 있다.

내가 만들었거나 다른 사람들이 만든 클래스를 사용하는데 close()가 필요하다면 `AutoCloseable`를 추가하자.

### 사용 예시

내가 사용할(close()가 필요한) 리소스를 try의 파라미터로 넣는다. 그럼 `AutoCloseable`이 구현되어 있다면 try문이 종료되면서 close()가 실행된다.

```java
public class Foo {
    public String foo() {
        try (BufferedReader br = new BufferedReader(new InputStreamReader(System.in))) {
            return br.readLine();
        }
    }
}
```

이 방식은 기존 try-finally가 가진 문제를 모두 해결할 수 있다.

가독성이 더 좋아지고 `close()`를 자동으로 해주기 때문에 성능에도 좋다.

에러를 추적하기 어렵다는 문제도 해결할 수 있다.

```java
public class MyResource implements AutoCloseable {
    public void doSomething() throws FirstError {
        System.out.println("doing something");
        throw new FirstError();
    }

    @Override
    public void close() throws SecondException {
        System.out.println("clean my resource");
        throw new SecondError();
    }
}

public class FirstError extends RuntimeException {
}

public class SecondError extends RuntimeException {
}

public class AppRunner {
    public static void main(String[] args) {
        try (MyResource myResource = new MyResource()) {
            myResource.doSomething();
        }
    }
}
```

문제가 되었던 코드를 tyr-with-resources 방식으로 구현했다. 

이렇게 하면 `doSomething()`에서 에러가 발생하면 `close()`에서 발생한 에러는 숨겨지고 `doSomething()`에서 발생한 에러를 확인할 수 있다.

그리고 `close()`에서 발생한 에러는 스택 추적 내역에서 `suppressed`를 달고 출력된다.

`Throwable.getSuppressed()`를 사용하면 코드에서도 바로 확인할 수 있다.

# 정리

`close()`를 해야하는 클래스는 무조건 `try-with-resources`를 사용하자!

JAVA에서도 IDE에서도 그렇게 하라고 하고있다!
