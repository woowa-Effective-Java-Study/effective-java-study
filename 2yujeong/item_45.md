# 아이템 45 – 스트림 사용 시 주의할 점

## 스트림
다량의 데이터 처리 작업을 돕고자 자바 8에 추가된 API
- stream : 데이터 원소의 시퀀스
- stream pipeline : 스트림 원소들로 수행하는 연산 단계
  - 소스 스트림 -> 중간 연산 -> 종단 연산
  - 중간 연산 : 한 스트림을 다른 스트림으로 변환
  - 종단 연산 : 중간 연산이 반환하는 스트림으로 최후의 연산 수행
    - ex) 원소를 정렬해 컬렉션에 담기, 특정 원소 선택, 모든 원소 출력 등
  - 지연 평가 : 스트림 파이프라인의 평가는 종단 연산이 호출될 때 이루어진다. 즉 종단 연산이 없으면 스트림 파이프라인은 아무 일도 하지 않게 된다.
  - 플루언트 API : 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성 가능한 API
- 장점 : 짧고 깔끔한 프로그램
- 단점 : 한 눈에 이해하기 어려운 가독성, 힘든 유지보수
<br>

## 코드 예시
### 아나그램
- 아나그램 : 철자를 구성하는 알파벳이 같고 순서만 다른 단어
  - A : babc, alphabetized(A) = abbc
  - B : cabb, alphabetized(B) = abbc
  - alphabetized(A) == alphabetized(B)이므로 A와 B는 아나그램 성립
- 사용자 입력 파일(dictionary)에서 단어를 읽어 사용자가 지정한 문턱값(minGroupSize)보다 원소 수가 많은 아나그램Set을 출력하는 코드 예시
1. computeIfAbsent
```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word)),
                    (unused) -> new TreeSet<>()).add(word);
            }
        }

        for (Set<String> group : groups.values()) {
            if (group.size() >= minGroupSize) {
                System.out.println(group.size() + ": " + group);
            }
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
- computeIfAbsent : 맵 안에 첫 번째 매개변수를 키 값으로 갖는 원소가 있다면 그 키에 매핑되는 값을 반환한다. 해당되는 원소가 없다면 두 번째 매개변수로 주어진 함수 객체를 키에 적용하여 값을 구한 뒤, 키와 계산된 값을 매핑하고 값을 반환한다.
2. 스트림 활용
```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                .values().stream()
                .filter(group -> group.size() >= minGroupSize)
                .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
- 스트림을 적절하게 활용하면 깔끔하게 구현할 수 있지만 너무 과용하여 가독성을 잃지 않도록 주의해야 한다.
<br>

## 스트림을 이용한 char 값 처리
- 자바에서 기본 타입인 char용 스트림은 지원 X
- char 값에 스트림을 적용하기 위해선 명시적인 형변환 등의 추가적인 처리가 필요
```java
"Hello World!".chars().forEach(x -> System.out.println((char) x));
```
**-> char 값들을 처리할 때는 스트림을 삼가는 편이 낫다.**

<br>

## 스트림(람다) vs 코드 블록
되풀이되는 계산에 대해 스트림은 주로 람다를 이용하여 표현하지만 반복 코드에서는 코드 블록을 이용하여 표현한다.
- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있지만 람다에서는 지역 변수 수정이 불가능하다.
- 코드 블록에서는 return, break, continue 등을 이용한 반복문 제어가 가능하며 예외를 던질 수 있지만 람다는 이중 어떤 것도 가능하지 않다.
### 스트림 사용이 유리한 경우
- 원소들을 일관되게 변환하는 경우
- 원소들을 필터링(특정 조건을 만족하는 원소 찾기)하는 경우
- 원소들을 하나의 연산을 사용해 결합하는 경우
- 원소들을 컬렉션에 모으는 경우
<br>

## 스트림으로 처리하기 어려운 경우
- 한 데이터가 파이프라인의 여러 단계를 통과할 때(여러 번의 중간 연산을 거칠 때), 각 단계에서의 변환 값들에 동시에 접근하기 어렵다.
- 앞 단계의 값이 필요할 때는 매핑을 거꾸로 수행하는 방법을 이용하는 게 바람직하다.
### 예시 – 메르센 소수 출력 프로그램
- 메르센 수 : 2^p – 1 형태인 수
- 메르센 소수 : p가 소수이면서 그 자체로도 소수인 메르센 수
- 처음 20개의 메르센 소수를 출력하는 코드 예시
```java
public static void main(String[] args) {
    primes().map(p -> TWO.pow(p.intValueExact()).substract(ONE))
        .filter(mersenne -> mersenne.isProbablePrime(50))
        .limit(20)
        .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
    return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```
- 각 메르센 소수의 앞에 지수(p)를 출력하길 원하는 경우 
  - p는 초기 스트림에만 나타나므로 종단 연산에서는 접근 불가 -> 중간 연산에서 수행한 매핑을 거꾸로 수행함으로써 p 계산 가능
```java
.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
```
<br>

## 스트림 vs 반복
### 카드 덱 초기화 프로그램 - 데카르트 곱
- 카드 덱을 초기화하기 위해선 숫자-무늬 쌍으로 만들 수 있는 모든 조합에 대한 계산 필요 -> 데카르트 곱
1. 2중 for문 활용
```java
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for (Suit suit : Suit.values()) {
        for (Rank rank : Rnak.values()) {
            result.add(new Card(suit, rank));
        }
    }
    return result;
}
```
2. 스트림 활용
```java
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
        .flatMap(suit ->
            Stream.of(Rank.values())
                .map(rank -> new Card(suit, rank)))
        .collect(toList());
}
```
<br>

## 결론
스트림과 반복 방식 중 어느 방식을 활용할지는 상황마다 가독성과 유지보수성을 고려하여 더 나은 방법을 찾는 것이 좋다.
