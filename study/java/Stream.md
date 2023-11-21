# 스트림 활용

### Predicate

filter 메서드를 사용해서 스트림의 요소를 조건에 맞게 걸러낼 수 있다.

```java
menu.stream()
    // 스트림의 요소 중 하나 씩 걸러낸다.
    .filter(Dish::isVegetarian)
    .collect(toList())
```

filter는 중간연산으로 스트림의 요소를 소모하지않는다.

조건에 맞는 요소를 걸러내 새로운 스트림을 반환한다.

### distinct

distinct는 중복된 요소를 걸러서 새로운 스트림을 반환하는 중간연산이다.

고유 요소를 식별해내는 건데 여기서 중요한 점은

Integer나 String 같이 이미 java에서 equals와 hashcode를 정의해놓은 객체는 따로 정의할 필요가 없지만

직접 만든 클래스라면 따로 적절히 정의해주어야한다.

### takeWhile()

자바의 정석에서도 본 기억이 없는 메서드다.

jdk 9에서 추가된 메서드인데 **이미 정렬된 스트림**에 적용하기 좋다.

기존 filter는 모든 요소를 다 돌면서 조건을 체크한다.

하지만 takeWhile()은 이미 정렬돼 있다는 것을 전제로 프리디케이트 문이 처음으로 거짓이 나오기 바로 직전까지만 스트림을 꺼낸다.

즉 데이터의 양이 매우 많다면 성능에 차이가 많이 난다.

### dropWhile()

takeWhile과 정반대의 역할을 한다.

처음으로 거짓이 되는 지점까지 모두 버린다.

즉 거짓만을 반환한다.

이 메서드 또한 정렬이 전제돼있어야한다.

### limit , skip

요소를 제한하거나 건너 뛸 수 있다.

### map

정말 정말 많이 사용하는 stream의 api중 하나다.

보통의 스트림 중간연산의 경우 본래의 스트림 타입과 같다.

```java
menu.stream()  // Stream<Dish>
    .filter() // Stream<Dish>
    .limit()  // Stream<Dish>
    .collect()
```

하지만 map은 본래의 스트림과 다른 타입의 스트림을 반환한다.

```java
menu.stream()  // Stream<Dish>
    .filter() // Stream<Dish>
    .map(Dish::getName)  // Stream<String>
    .limit()  // Stream<String>
    .collect()
```

### flatMap

한 번도 사용은 해보지 않았지만 나름 사용할 용도가 있어 보이긴 한다.

문자열 배열의 알파벳들 중 유니크한 알파벳만 찾고싶다.

["hello","world"]

를 스트림으로 생성하면 Stream<String>이다.

각 요소가 String 이기 때문이다.

여기서 map을 이용해 각 요소를 알파벳단위로 자르면 Stream<String[]>로 반환한다.

여기서 distinct()를 사용하면 원하는 결과를 얻을 수 없다.

h e l l o 가 하나의 요소이고 w o r l d 가 하나의 요소이기때문에 두 요소는 정확히 다른 요소이기 때문이다.

각 알파벳을 하나하나의 요소로 만들어야만 한다.

해결법은 각 문자열 배열을 하나의 스트림으로 만들어 총 2개의 스트림으로 생성한 뒤에 하나의 스트림으로 합치면 된다.

이때 flatMap(Arrays::stream)을 사용한다.

### 퀴즈 5-2

각 요소의 제곱근 반환

```java
public class Main {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1,2,3,4,5);

        List<Double> collect = list.stream()
                .map(s -> Math.pow(s,2))
                .collect(Collectors.toList());

        System.out.println(collect);
    }
}
```

[1,2,3] [3,4]를

[(1,3),(1,4),(2,3)(2,4)(3,3)(3,4)] 로

```java
public class Main {
    public static void main(String[] args) {
        List<Integer >list1 = Arrays.asList(1,2,3);
        List<Integer >list2 = Arrays.asList(3,4);

        List<int[]> collect = list1.stream()
                .flatMap(i -> list2.stream()
                        .map(j -> new int[]{i, j}))
                .collect(Collectors.toList());
        System.out.println(collect);
    }
}
```

위에서 합이 3으로 나누어떨어지는 쌍만

```java
public class Main {
    public static void main(String[] args) {
        List<Integer >list1 = Arrays.asList(1,2,3);
        List<Integer >list2 = Arrays.asList(3,4);

        List<int[]> collect = list1.stream()
                .flatMap(i -> list2.stream()
                        .map(j -> new int[]{i, j}))
                .filter(i->(i[0]+i[1])%3 == 0)
                .toList();

        for (int i = 0; i < collect.size(); i++) {
            System.out.println(Arrays.toString(collect.get(i)));
        }
    }
}
```

## 검색과 매칭

allMatch : 모든 요소가 프리디케이트에대해 참인지 거짓인지를 반환한다. (최종연산)

anyMatch : 적어도 하나의 일치하는 요소가 있는지 확인

noneMatch : 일치하는 요소가 없는지 확인

이 메서드들은 **쇼트서킷**기법이다.

and로 연결된 아주 커다란 연산이 있을 때 앞쪽에서 하나라도 거짓이 나오면 모두 거짓이듯이

모든 스트림의 요소를 소모하지않아도 판별할 수 있는 기법이다.

무한 스트림도 이러한 기법을 사용해 유한한 요소처럼 사용할 수 있다.

## 요소 검색

findAny는 요소 중 일치하는 임의의 요소 하나만을 반환한다.

만약 일치하는 요소가 없다면 어떻게될까?

Null일까?

Null 이긴하다.

하지만 Null을 좀 더 안전하게 사용하기위해 Optional<>로 감싸서 반환한다.

Optional은 10장에서...

### findFirst findAny

findFirst 는 찾아낸 첫 요소를 반환한다.

findAny와 비교하면 차이점은 바로 순서다.

왜 두 가지 모두 다 있는지 이유를 궁금해하지도 않았지만

알게됐다.

바로 병렬처리때문인데 병렬 처리는 첫 요소를 알 수가 없다.

작업이 나뉘어 동시처리되기때문에 누가 먼저 찾은 값이 처음인지를 알 수 없기때문이다.

## reduce

reduce는 2가지 파리미터를 받는다.

첫 번째는 초깃값

두 번째는 BinaryOperator ex) (a,b) -> a+b;

초깃값은 생략이 가능하다. 하지만 아무런 요소가 없는 스트림에 적용하면 반환할 값이 없기때문에 Optional로 반환된다.

reduce는 누적합을 구하기에 매우 적합하다.

요소를 순회하면서 초깃값에 요소들을 더해 누적해서 값을 반환한다.

### 최솟값 최댓값

reduce는 최대 최소를 찾기에도 적합하다.

numbers.stream().reduce(Integer::min);

위 코드가 가능한 이유는

numbers.stream().reduce((i,j) -> Integer.min(i,j))

> Integer.min(i,j)는 i와j중 작은 값을 반환

결국 BinaryOperator 형식과 똑같기 때문에 가능한 것이다.

### reduce의 장점

스트림을 사용하지않고 누적합을 구하기위해선

sum += array[i]

를 루프를 돌려서 구할 것이다.

여기서 sum은 누적합을 담을 변수인데

위 작업을 병렬처리하기위해선 fork/join framework 방식을 사용해야한다.

작업을 둘로 나누고 각자의 작업이 완료되면 합치는 방식이다.

reduce 패턴은 위처럼 외부반복이 아닌 내부반복으로 추상화했기때문에 쉽게 병렬 처리가 가능하다.

> 왜 내부반복이 병렬처리를 쉽게할 수 있는지는 잘 모르겠다.

### 내부 상태의 유지

filter는 직전 상태가 필요할까?

필요 없다. 새로운 요소가 조건에 부합하는지만 알면 된다.

반면 distinct나 reduce는 직전 상태를 알아야한다.

중복된 요소나 직전 요소의 상태를 알아야지만 다음 연산이 의미가 있기때문이다.

이를 상태 없음과 상태 있음으로 나눈다.
