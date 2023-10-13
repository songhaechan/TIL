## 중간연산과 최종연산

중간연산은 filter, map, distinct 같은 메서드들은 반환값이 다시 stream이다.

> Method Chain으로 연결해서 사용이 가능하다.

최종연산은 foreach, collect, reduce는 요소를 다 소모하면 스트림이 닫혀 재사용이 불가능하다.

스트림은 원본을 변형시키지 않기때문에 똑같은 스트림을 다시 만들 수 있다.

또한 스트림의 중간연산은 호출 시 바로 실행되지않고 최종연산이 호출되면 그 시점에 중간연산을 실행한다.(명시만 해줄 뿐이다.)

## 스트림의 오토박싱, 언박싱

여타 Wrapper 클래스들과 비슷하게 기본형타입과 참조형 타입으로 오토박싱, 언박싱이 일어나기때문에 일반 스트림 대신 IntStream과 같은 기본형 스트림을 사용하는 것이 성능상 좋다.

## map 함수

개인적으로 가장 많이 사용했었고 가장 중요하다고 생각하는 함수다.

요소들을 다른 타입으로 변형하는 경우에 사용한다.

Spring에서는 Entity를 Dto로 변환할 경우 필드에 컬렉션 타입이 있을 경우 그 타입들도 모두 dto로 변환해서 entity의 외부 노출을 모두 막아야하는데 그 경우 매우 유용하게 사용할 수 있다.

### 기본형 스트림으로 반환하는 map함수

mapToInt, mapToLong 등 Stream<T>타입으로 반환하지않고 IntStream, LongStream으로 반환할 수 있다.

### flatMap

요소가 모두 단일 객체이거나 단일 기본형 타입일 수는 없다.

요소자체가 배열이고 요소들을 하나의 스트림으로 만들고 싶다면 map을 사용해서는 안된다.

map은 배열 자체를 스트림으로 생성해서 Stream<Stream<T>>형식으로 반환한다.

flatMap은 Stream<T>형식으로 반환한다.

### Collect

중간연산을 마치고 요소들을 어떤 컬렉션으로 반환해야할지를 결정하는 함수다.

Collect(Collectors.toList) 와 같이 리스트 타입으로 반환할 수 있다.

> 다른 타입도 가능하다. 배열, Set, Map ...

## Optional

### Optional 바르게 사용하기

https://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/

### Optional은 자바의 널 안정성이 낮다는 비판에 나온 Wrapper객체이다.

## NPE 예방과 코드 가독성 향상

기본적으로 코드를 작성하면 NPE를 예방하기위해서 if문을 통해 널체크를 진행한다.

하지만 이 작업을 좀 더 간결하게 만들 수 있다. (feat Lamda & Stream)

## get() 쓰지말자.

Optional이 등장한 배경을 아예 무시하는 메서드다.

get으로 직접 포장지를 벗겨서 사용하는건 좋지않다.

NPE예방에도 전혀 도움이 되지않는다.

orElse()를 실제로 더 많이 사용하고 안전하다.

> orElse(), orElseThrow(), orElseGet() 애용

## Optional은 비용이 비싸다 너무 남발하진 말자.

Optional은 여타 다른 Wrapper 객체들이 그러하듯(Integer, Long 등) 생성비용이 있다.

모든 객체를 다 Optional로 두르는건 비효율적이다.

필요한 경우에 사용하자.

