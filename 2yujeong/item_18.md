# item 18 - 상속보다는 컴포지션을 사용하라

#### 화면에 출력되는 배카라의 키는😧?
```java
class PhysicalInfo {
    public void baekara() {
        printHeight();
    }

    public void printHeight() {
        System.out.println(174);
    }
}

class RealPhysicalInfo extends PhysicalInfo {
    @Override
    public void baekara() {
        super.baekara();
    }

    @Override
    public void printHeight() {
        System.out.println(165);
    }
}
```
```java
public static void main(String[] args) {
    PhysicalInfo crew = new RealPhysicalInfo();
    crew.baekara();
}
```

## Self-Use
메소드가 자신과 동일한 클래스 내의 다른 메소드를 사용하는 것<br>
이때 사용하는 메소드가 재정의 가능한 메소드라면 상속으로 인해 예상하지 못한 오류가 발생할 수 있다.

<br>

## 상속은 캡슐화를 깨트린다.
상위 클래스의 구현 방식에 따라 하위 클래스의 동작이 영향을 받을 수 있다.

#### 화면에 출력되는 _진짜_ 키는😶?
```java
class PhysicalInfo {
    public int height = 155;

    public void increaseHeight() {
        height++;
    }

    public void getTaller(int length) {
        for (int i = 0; i < length; i++) {
            increaseHeight();
        }
    }
}

class RealPhysicalInfo extends PhysicalInfo {
    public int realHeight = 155;

    @Override
    public void increaseHeight() {
        realHeight++;
        super.increaseHeight();
    }

    @Override
    public void getTaller(int length) {
        realHeight += length;
        super.getTaller(length);
    }
}
```
```java
public static void main(String[] args) {
    RealPhysicalInfo eve = new RealPhysicalInfo();
    eve.getTaller(3);
    System.out.println(eve.realHeight);
}
```
#### -> Self-Use로 인해 코드가 예상했던 대로 동작하지 않는다.

### 해결 방법 1 - getTaller 재정의 메소드 제거
- 상위 클래스의 getTaller 메소드가 increaseHeight를 이용하여 구현했을 경우에만 성립하는 해법
- 구현 방식이 바뀌면 올바른 작동 보장 X

### 해결 방법 2 - getTaller를 다른 방식으로 재정의
```java
class PhysicalInfo {
    public int height = 155;
    private boolean isUpdated = false;

    public void increaseHeight() {
        height++;
    }

    public void getTaller(int length) {
        isUpdated = true;
        for (int i = 0; i < length; i++) {
            increaseHeight();
        }
    }
}

class RealPhysicalInfo extends PhysicalInfo {
    public int realHeight = 155;

    @Override
    public void increaseHeight() {
        realHeight++;
        super.increaseHeight();
    }

    @Override
    public void getTaller(int length) {
        for (int i = 0; i < length; i++) {
            increaseHeight();
        }
    }
}
```
- 상위 클래스의 getTaller 메소드의 구현 방식과는 관련 없이 항상 제대로 동작하지만 코드 중복이 발생하며 성능을 떨어뜨릴 수 있다.
- 만약 상위 클래스의 getTaller 메소드에서 하위 클래스에서는 접근할 수 없는 private 필드를 변경하는 코드를 가지고 있다면 이 방법으로는 원하는 대로 동작할 수 없다.

### 해결 방법 3 - 메소드 재정의 회피
- 메소드를 재정의하는 대신 새로운 메소드를 추가하는 방법
- 해결 방법 1, 2보다 안전하지만 나중에 상위 클래스에 추가되는 새 메소드명과 하위 클래스에서 재정의를 피하기위해 선언했던 메소드명이 겹치는 경우가 발생할 수 있다.
<br>

## 상속보다는 컴포지션
기존 클래스를 상속하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스를 참조하는 컴포지션을 활용하면 상속으로 인한 문제점을 방지할 수 있다.
```java
class RealPhysicalInfo extends PhysicalInfo {
    private PhysicalInfo physicalInfo;
    public int realHeight = 155;

    public RealPhysicalInfo() {
        physicalInfo = new PhysicalInfo();
    }

    @Override
    public void increaseHeight() {
        realHeight++;
        physicalInfo.increaseHeight();
    }

    @Override
    public void getTaller(int length) {
        realHeight += length;
        physicalInfo.getTaller(length);
    }
}
```
<br>

## 결론
- 상속을 함부로 사용했다간 예상하지 못한 오류를 맞닥뜨리게 될 수 있다.
- 상속보다는 컴포지션을 활용하자.
- 배카라 키 165
