Mokito
=

우선 모킹을 알아보면서 객체지향적이지못한 설계가 주는 단점을 하나 발견했다.

```java
private final CarDao carDao = CarDao.getInstance();
```

기존 CarService클래스는 CarDao를 위와같이 사용하고있었다.

구체클래스에 직접적으로 의존하는 형태다.

그리곤 아래와같이 테스트코드를 작성하려했다.

```java
@ExtendWith(MockitoExtension.class)
class CarServiceTest {

    @Mock
    CarDao carDao;
    CarService carService = new CarService();

    @BeforeEach

    @Test
    @DisplayName("차량등록 후 차량조회 시 반환되는 PK가 같아야한다.")
    void 차량_등록() {
        //given
        CarRegisterDto dto = CarRegisterDto.builder()
                .carId(1L).build();
        Car car = Car.of(dto);
        //when
        when(carDao.findById(1L)).thenReturn(Optional.of(car));
        Car findCar = carService.findCar(1L);
        //then
        Assertions.assertThat(car.getCarId()).isEqualTo(findCar.getCarId());
    }

}
```

우선 build.gradle에 의존성을 추가했다.

그 후에 테스트클래스레벨에 @ExtendWith(MockitoExtension.class) 어노테이션을 붙여 mokito를 가져왔다.

@Mock 어노테이션을 사용하면 객체를 직접생성하지도 않았지만 객체를 사용할 수 있게됐다.(아무래도 리플렉션의 결과일것 같다.)

그 후에 when()메서드안에 내가 원하는 메서드가 강제로 반환할 인스턴스를 넘기면 실제 그 메서드가 실행되면 내가 기대하는 객체가 나오는 메커니즘이다.

이 외에도 예외를 던지게해주고 어떤 인수로 실행되는지 모를경우 파라미터에 any()로 넘기면 똑같이 원하는 인스턴스를 반환받을 수 있다.

**하지만 문제가 생겼다**

### 테스트가 완벽하게 실패했다

원인은 CarService가 Mocking한 Dao를 사용하지않고 기존 Dao를 사용했기때문이다. (DB에 INSERT쿼리가 날아가지않았기때문에 조회가 불가능하다는 에러)

당연히 CarService는 모킹한 객체에 의존해야하기에 생성자를 이용해 주입하는 방식을 택해야했다.

```java
@Mock
    CarDao carDao;
    CarService carService = new CarService(carDao);
```

```java
private final CarDao carDao;
    public CarService(CarDao carDao) {
        this.carDao = carDao;
    }
```

이렇게 생성자 주입을 통해 모킹할 객체를 주입받도록 할 수 있다.

### 그런데 테스트는 또 실패한다.

CarService인스턴스를 사용하는 모든 곳의 생성자에 문제가 생겼다.

인자 개수에 0을 기대하지만 1이 있다는 에러다.

### 결론은 생성자를 통해 의존관계를 주입하지않고 구체클래스를 이용했기때문에 DIP를 위반했기때문이다.

사실 CarService에서 사용하는 CarDao는 절대 변하지 않을 것이란 믿음에서 구체클래스를 작성했다.

앞으로 모킹과 확장성있는 설계를 위해 생성자주입을 사용하자...