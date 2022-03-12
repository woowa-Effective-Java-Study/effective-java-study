# 아이템 46 - 부작용 없는 함수를 이용한 올바른 스트림 사용

## 바람직하지 않은 스트림 사용 예시
```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```
- freq라는 외부 상태를 스트림 내의 람다에서 변경 -> side effect 발생
- forEach는 기존의 반복적 코드와 별다를 바 없으며 병렬화 할 수 없다.

=> **스트림 계산 결과를 보고할 때만 쓰고 계산하는 데는 쓰지 말자**

<br>

## Collertor 인터페이스
스트림 API를 올바르게 사용하기 위해서는 Collector에 대해 잘 알아두는 게 좋다. Collector를 활용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다.
```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words
        .collect(groupingBy(String::toLowreCase, counting()));
}
```
- side effect가 발생하지 않는다. 

=> forEach 대신 java.util.stream.Collertors 클래스를 활용하자

### Collector를 이용하여 freq에서 가장 흔한 단어 10개를 뽑아내는 코드
```java
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
```
- Collector를 이용하여 스트림에서 원소를 추출하여 리스트에 담는 코드를 쉽게 구현할 수 있다.
<br>

## Map Collector
Collectors 클래스에 구현된 메소드들 중 대부분은 스트림을 맵으로 취합하는 기능을 한다.

### 가장 간단한 Map Collector - toMap(keyMapper, valueMapper)
- 스트림 원소를 키에 매핑하는 함수 객체와 값에 매핑하는 함수 객체를 인수로 받아 map을 반환한다,
```java
private static final Map<String, Operation> stringToEnum =
    Stream.of(values()).collect(
        toMap(Object::toString, e -> e)
    );
```

### 매개변수 3개를 받는 toMap
- 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때 유용하다.
```java
Map<Artist, Album> topHits = albums.collect(
    toMap(Album::artist, a -> a, maxBy(comparing(Album::sales)))
);
```

### 매개변수 4개를 받는 toMap
- EnumMap이나 TreeMap처럼 원하는 특정 맵 구현체를 직접 지정할 수 있다.
<br>

## Collertors - groupingBy
- groupingBy : 분류 함수를 인수로 받아 원소들을 카테고리별로 모아놓은 맵을 반환한다.
  - 카테고리가 맵의 키가 되며 각 키에 속하는 스트림의 원소들을 담은 리스트가 값이 된다.
```java
words.collect(groupingBy(word -> alphabetize(word)))
```
- 값의 형태를 리스트가 아닌 다른 자료구조가 되게 하려면 toSet(), counting() 등의 downstream collector를 인수로 넣어주면 된다.
  - downstream collector : 첫 번째 그룹의 결과를 기준으로 2차 그룹화를 수행한다.
    - toSet() : 원소들의 집합을 값으로 갖는 맵을 반환한다.
    - toCollection(collectionFactory) - 원하는 컬렉션 타입을 선택할 수 있으므로 유연하다.
    - counting() - 각 키에 해당하는 원소들의 개수가 값으로 매핑된다.
<br>

## Collectors - minBy, maxBy
인수로 받은 비교자를 이용하여 스트림에서 값이 가장 작은 혹은 가장 큰 원소를 찾아 반환한다.

<br>

## Collectors - joining
원소들을 연결하는 Collector를 반환하며 문자열 같은 CharSequence 형태의 인스턴스의 스트림에만 적용 가능하다.
- 매개변수 0개 : 단순히 원소들을 연결하는 Collector 반환
- 매개변수 1개 : delimiter를 매개변수로 받아 원소들의 연결 부위마다 구분자를 삽입
- 매개변수 3개 : prefix, delimiter, suffix를 받아 원소들의 맨앞 / 각 연결부 / 맨뒤에 삽입
<br>

## 결론
스트림을 올바르게 사용하기 위해선 Collector를 잘 알아두는 게 좋다.
