# item 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## public 필드를 사용한다는 의미

```java
import java.util.Collections;

public class Car {
    public String name;
    public int position;
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.name = "붕붕이";
        car.position = 0;

        // 앞으로 한칸 전진
        car.position++;
    }
}
```

### 단점!

- 클래스의 데이터 필드에 직접 접근할 수 있기 때문에 캡슐화의 이점을 제공하지 못한다.
    - 캡슐화
        - 유사한 기능이나 변수를 한 집합으로 하여 더 관리하기 쉽게하고 코드를 명확하도록 함
        - 외부에서 멤버를 액세스하는 방법을 지정할 수 있도록, 직접 접근을 막거나 접근 전 부가적인 처리를 요구
        - 정보를 외부에 은닉하여 외부에서는 해당 집합의 세부 내용에 집중하지 않도록 함
- API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
- 불변식을 보장할 수 없다.
    - 값을 그냥 바꿀 수 있기 때문에
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.
    - 1차원적인 데이터 접근만이 가능하고 추가 로직을 추가할 수 없다.
- 무결성을 보장할 수 없다.
- 데이터가 언제 어디서나 변할 수 있기 때문에 데이터를 활용할 때 방어적 복사를 해야하고 성능 문제가 생긴다.
    - 얕은 복사 : List a = b; // 모든 주소 공유
    - 방어적 복사 : List A = Collections.unmodifiableList(B); // 객체 주소 다름, 내부 요소들은 주소 공유
    - 깊은 복사 : List a = new ArrayList(b); // 모든 주소 다름

---

### 최소한의 캡슐화라도 한 클래스

```java
public class Car {
    private String name;
    private int position;

    public PrivateCar(String name) {
        this.name = name;
        this.position = 0;
    }

    public void moveForward() {
        position++;
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();

        // 앞으로 한칸 전진
        car.moveForward();
    }
}
```

---

### 불변 필드를 노출한 public 클래스라면 public 필드를 써도 될까?

```java
public class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;
    
    public final int hour;
    public final int minute;

    public Time(final int hour, final int minute) {
        this.hour = hour;
        this.minute = minute;
    }   
}
```

불변식이 보장된다는 것 외에는 단점이 해결되지 않는다.
- 캡슐화 문제
- 부수 작업 못하는 문제

---

### private 중첩 클래스

```java
public class MathGrades {
    static class Grades {
        public double value;
    }
    
    public Grades getTopGrades() {
        Grades grades = new Grades();
        grades.value = 4.5;
        return grades;
    }
}
```

- 직접 데이터 접근 불가(아래의 문제점 어느정도 해결 가능)
  - 캡슐화 문제
  - 내부 표현 수정 문제
  - 부수 작업 못하는 문제
  - 무결성 보장 문제
- private으로 하면 같은 클래스만 사용하기 때문에 상관없음
- package-private으로 하면 같은 패키지만 사용하기 때문에 잘 관리하면 좋을 수도 있음
- 객체 지향적으로 구현 가능

## 요약

public 클래스는 가변 필드를 public으로 노출하지 말자
-> 불변 필드라고 노출시켜도 되는건 아니다
private, package-private 클래스인 경우 노출하는 편이 나을 때도 있다고 하지만 가능하면 쓰지말자
