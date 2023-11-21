## NPE

아래 코드를 보자.

```java
public class Main {
    public static void main(String[] args) {
        Car car = null;
        Person p = new Person(car);
        p.getCar().getInsurance().getName();
    }
    static class Person{
        private Car car;
        public Car getCar(){return car;}

        public Person(Car car) {
            this.car = car;
        }
    }
    static class Car{
        private Insurance insurance;
        public Insurance getInsurance(){return insurance;}
    }
    static class Insurance{
        private String name;
        public String getName(){return name;}
    }
}

```

Person 객체 안에 Car 안에 Insurance 안에 String 데이터가 중첩구조로 들어가있다.

그런데 만약 차가 없는 사람의 자차보험을 조회하려하면

```java
p.getCar().getInsurance().getName();
```

은 NPE가 발생한다.

그렇다면 null체크를 해야한다.

```java
public String getCarInsuranceName(Person person){
        if(person != null){
            Car car = person.getCar();
            if(car != null){
                Insurance insurance = car.getInsurance();
                if(insurance != null){
                    return insurance.getName();
                }
            }
        }
    }
```

이렇게 널체크를 해주어야한다.

위 코드가 보기 좋은 코드는 아닐 뿐더러 유지보수하기도 쉽지않다.

> 들여쓰기가 늘어나면 늘어날 수록 가독성은 떨어진다.

더 큰 문제는 null확인을 깜빡이라도한다면 NPE는 더욱 쉽게 발생한다.

## Null의 문제

- Null 체크 코드가 늘어난다 + 가독성이 떨어진다.
- Null은 아무 의미가 없다.
- NPE의 근원이다.
- null은 의미가 없는데, 의미를 부여한 null(이런 경우가 있을 수 있음)은 해석하기 어렵다.

## Optional

정말 단순하게 값이 있을 수도 없을 수도 있는 선택형 객체다.

```java
static class Person{
        private Optional<Car> car;
        public Optional<Car> getCar(){return car;}
    }
```

이렇게 어떤 사람은 차가 있을 수도 있고 없을 수도 있음을 나타낸다.

메서드의 이름만 보고도 이 메서드가 반환하는 값이 선택형이라는 것을 쉽게 알 수 있다.

```java
public Optional<Car> getCar(){return car;}
```

하지만 보험회사의 이름은 조금 다르다.

```java
static class Insurance{
        private String name;
        public String getName(){return name;}
    }
```

보험회사는 이름이 없어서는 안된다.

실수에 의해 null이 들어왔다하더라도 이건 찾아내서 해결해야할 문제이지 **무작정 Optional로 감싸서는 안된다.**

## Optional 만들기

```java
Optional<Car> car = Optional.of(car);
```

of는 null이 아닌 값을 포함하는 Optional을 만든다.


```java
Optional<Car> car = Optional.ofNullable(car);
```

ofNullable은 null이면 빈 Optional 객체를 반환한다.

Optional.empty()와 같다.

### 옵셔널 객체에 바로 get() 쓰지말자 null을 사용할 때와 같은 문제가 발생한다.

## Optional.map()

스트림과 비슷하게 map을 제공한다.

애초에 Optional은 요소가 1개인 스트림이라고 생각하면 좋다.

```java
Optional<Insurance> i = Optional.ofNullable(insurance);

Optional<String> name = i.map(Insurance::getName);
```

**보험안의 이름을 안전하게 map으로 꺼내는 코드다.**

그런데 만약 name이 없다면?

orElse로 정해진 이름을 반환하던지 orElseThrow로 예외를 발생시키던지 후에 처리를 해주면된다.

```java
Optional<Insurance> insurance = Optional.ofNullable(insurance);

Optional<String> name = isurance.map(Insurance::getName);
```

## flatMap()

스트림과 마찬가지로 Optional도 중첩되면 Optional<Optional<Car>> 식으로 겹겹이 둘러 쌓일 수 있다.

이때 똑같이 flatMap()을 사용하면 평준화된다.

```java
Optional<Person> p = Optional.of(person);

p.map(Person::getCar) // Optional<Optional<Car>>
 .map(Car::getInsurance)// Optional<Optional<Optional<Insurance>>>
 .map(Insurance::getName)// Optional<Optional<Optional<Optional<String>>>>

```

그냥 맵을 사용하면 안에 있는 값자체도 Optional이기때문에 계속 중첩된다.

```java
Optional<Person> p = Optional.of(person);

p.flatMap(Person::getCar)
 .flatMap(Car::getInsurance)// Optional<Car>
 .map(Insurance::getName)// Optional<String>
 .orElse("Unknown") // 이름이 없다면 기본값
```

## orElse()

get()을 사용하지말고 orElse()나 orElseThrow 혹은 ifPresent()를 사용해후 get()을 사용해야한다.

orElse는 값이 없을 때 반환할 객체를 말한다.

orElseThrow는 값이 없다면 지정한 예외를 터트릴 수 있다.

## Optional과 스트림 조작 (제일 많이 사용할 듯)

list에서 자동차를 소유한 사람들이 가입한 보험회사의 이름을 set으로 반환하는 비즈니스 요구사항이 있다고 가정하자.

```java
public Set<String> getCarInsuranceNames(List<Person> persons){
    return persons.stream()
                  .map(Person::getCar)
                  .map(c -> c.flatMap(Car::getInsurance))
                  .map(i -> i.map(Insurance::getName))
                  .flatMap(Optional::stream)
                  .collect(toSet());
}
```

```java
.map(Person::getCar)
// 첫 map은 바깥쪽 stream api의 map이다.
// 즉, Stream<List<Person>>을 Stream<List<Optional<Car>>> 로 변환
```

```java
.map(c -> c.flatMap(Car::getInsurance))
// 마찬가지로 바깥쪽은 stream api
// 안쪽 flatMap을 이용해 Stream<List<Optional<Insurance>>>
// 바깥쪽 map은 Stream<List<Optional<Car>>> -> Stream<List<Optional<Insurance>>>
```

```java
.map(i -> i.map(Insurance::getName))
// Insurance의 name필드는 Optional이 아니기때문에 그냥 map
// Stream<List<Optional<Insurance>>> -> Stream<List<Optional<String>>>
```

```java
.flatMap(Optional::stream)
// Optional을 stream화 하면 빈 optional은 제외하고 값이 있는 optional을 stream의 요소로 변환
// 그런데 이 때 flatMap을 안써주면 Stream<Stream<String>>이 되기때문에 flatMap으로 평준화해 Stream<String>으로 최종 변환
```

## 11-1 퀴즈

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(
            Optional<Person> person, Optional<Car> car
    ){
        return person.flatMap(p-> car.map(c->findCheapestInsurance(p,c)));
    }
```

첫 번째 flatMap에 들어가서 p가 존재한다면 람다식이 실행되지만 없다면 그대로 빈 optional이 반환된다.

값이 있다면 car의 map이 호출되고 car가 있다면 가장 싼 보험료를 찾는 메서드가 안전하게 호출된다.

Null 체킹 코드는 단 한 줄도 없다.

> flatMap을 사용안하고 map을 사용하면 메서드에서 원하는 반환타입이 나오지 않는다. (Optional<Optional<Insurance>> 가 나옴)

## 기본형 Optional 사용하지 말기

Optional은 기본형에 맞춰 OptionalInt, OptionalDouble, OptionalLong이 있다.

일단 기본형 스트림과는 달리 기본형 옵셔널은 flatmap, map, filter를 사용할 수 없다.

즉 범용성이 떨어진다. 일반 Optional과 혼용해서 쓰기가 어렵다는 말이다.

Optional은 비용이 든다는 것을 명심하고 남발하면 안된다.

