# 아이템 25 - 하나의 파일에는 하나의 Top Level 클래스만 담기

#### Main.java
```java
public static Main {
    public static void main(String[] args) {
        System.out.println("알린 친구 " + Alien.FRIEND + ", 이브 친구 " + Eve.FRIEND);
    }
}
```

#### Alien.java
```java:Alien.java
class Alien {
    static final String FRIEND = "Woody";
}

class Eve {
    static final String FRIEND = "Wall-E"
}
```

#### 💻 출력 결과
```
알린 친구 Woody, 이브 친구 Wall-E
```

#### Eve.java
```java
class Alien {
    static final String FRIEND = "Alex";
}

class Eve {
    static final String FRIEND = "Baekara";
}
```

<br>

### 컴파일 에러가 발생하는 경우
```
javac Main.java Eve.java
```
1. Main.java 파일 먼저 컴파일
    - main 메소드에서의 참조를 위해 Alien.java를 탐색
    - Eve.FRIEND보다 Alien.FRIEND가 먼저 나오므로 Eve.java 파일이 아닌 Alien.java 파일 탐색
    - 컴파일러에서 Alien.java 파일의 Alien 클래스와 Eve 클래스의 존재를 알게 된다.
3. 두 번째로 Eve.java 파일 컴파일
    - 앞서 탐색한 Alien.java 파일의 Alien 클래스, Eve 클래스와 중복 정의인 것을 알고 컴파일 에러를 발생시킨다.

<br>

### 컴파일 순서에 따라 결과가 달라지는 경우
```
javac Main.java
javac Main.java Alien.java
```
출력 결과 👉 `알린 친구 Woody, 이브 친구 Wall-E`
<br><br>

```
javac Eve.java Main.java
```
출력 결과 👉 `알린 친구 Alex, 이브 친구 Baekara`
<br><br>

**컴파일러에 건네지는 소스 파일의 순서에 따라 동작이 달라짐 -> 의도한 대로 동작하지 않을 수 있다.**
<br><br>

### 결론
한 파일에는 하나의 Top Level 클래스만 선언하자
