DTO ( Data Transfer Object )
=

![img](https://file.okky.kr/images/1665031857276.png)

## DTO란 계층간 데이터 전송을위해 도메인모델 대신 사용하는 객체다.

View에서 사용자의 요청이 들어오면 Presentation(Controller)이 Dto객체로 역직렬화하여 데이터를 받는다. (Json -> Java Object)

이 Dto객체가 Service단에 가서 요청한 로직을 처리하고 도메인 모델(Entity)로 변환한 후 DB에 접근해 요청을 처리한다.

이렇게 View - Controller - Service 계층간 사용되는 데이터의 전달을 Dto를 이용해 하게된다.

## 왜 DTO를 써야할까? Entity를 직접 쓰면 어떻게 될까?

### - 실제 Client가 입력/요청한 정보와 실제 DB에 저장된 데이터사이에는 차이가 있다.

아래 예시를 보면


도메인 모델(Entity)
```java
@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
//중고차 Entity
public class UsedCar {
    @Id @GeneratedValue
    private Long id;       //DB id
    private String color;  //색
    private Long mileage;  //주행거리
    private Long price;    //가격
    private String name;   //차종
    private String model_year; //연식
}
```

사용자는 id를 입력하지않았지만 내부에서 id값을 추가해 DB에 저장한다. id외에도 요청한 시간 등 다양한 부가정보를 덧붙여 DB에 저장한다.

Entity를 직접 사용한다고 생각해보자. Client는 필요없는 정보 혹은 보안상 민감한 정보까지도 전달받기때문에 이를 분리시키기위해 DTO와 Entity를 구분해서 사용한다.

### - View 와 Entity의 결합도

Dto를 사용하지않고 Entity를 사용하면 View와 Entity의 결합도가 강해진다. 

View의 변화가 Entity에 영향을 미치고 그 반대도 마찬가지이다.

이같은 결합도를 느슨하게 만들어주는것이 Dto다.

## Dto는 언제 변환해야할까?

> A Service Layer defines an application's boundary [Cockburn PloP] and its set of available operations from the perspective of interfacing client layers. It encapsulates the application's business logic, controlling transactions and coor-dinating responses in the implementation of its operations. 마틴 파울러 - Service Layer

마틴 파울러 아저씨는 Service Layer를 비즈니스 로직, 즉 도메인을 보호하는 레이어라고 설명한다.

그렇기때문에 Service는 도메인을 캡슐화하여 보호하는 것이고 Dto를 이용해 이러한 정의를 지킬 수 있다.

Mapper
=

위에서 Entity에서 Dto로 Dto에서 Entity로 데이터를 가공해야한다는 점을 언급했다.

Mapper는 Entity에서 Dto로 Dto에서 Entity로 변환하는 객체를 의미한다.

```java
@Component
public class UsedCardMapper {
    public UsedCar toEntity (UsedCarRequestDto dto) {
        return UsedCar.builder()
                .color(dto.getColor())
                .mileage(dto.getMileage())
                .price(dto.getPrice())
                .name(dto.getName())
                .model_year(dto.getModel_year())
                .build();
    }

    public UsedCarResponseDto toResponseDto (UsedCar entity){
        return UsedCarResponseDto.builder()
                .id(entity.getId())
                .color(entity.getColor())
                .mileage(entity.getMileage())
                .price(entity.getPrice())
                .name(entity.getName())
                .model_year(entity.getModel_year())
                .build();
    }
}
```
위처럼 Mapper는 DB에서 꺼내온 데이터를 ResponseDto로 변환하고 RequestDto를 Entity로 변환한다.

```java
@Service
public class UsedCardService {
    private UsedCardRepository usedCarRepository;
    private UsedCardMapper usedCarMapper;

    @Autowired
    public UsedCardService(UsedCardRepository usedCarRepository,UsedCardMapper usedCarMapper){
        this.usedCarRepository = usedCarRepository;
        this.usedCarMapper = usedCarMapper;
    }
    public UsedCarResponseDto getUsedCar(Long id){
        return usedCarMapper.toResponseDto(getEntity(id));
    }

    public Long saveUsedCar(UsedCarRequestDto usedCar){
        UsedCar entity = usedCarRepository.save(usedCarMapper.toEntity(usedCar));
        return entity.getId();
    }
}
```

Service Layer에서 Mapper객체를 의존객체주입을 하고 알맞은 변환을 하고 있다.

MapStruct
=
Map Struct는 Mapper객체를 생성해주는 외부 라이브러리다.

JPArepository에는 Hibernate 구현체가 있듯이 Map Struct는 우리가 생성하고싶은 Mapper객체를 생성해준다.

```java
implementation 'org.mapstruct:mapstruct:1.5.3.Final'
	annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.3.Final'
```
### 위 의존성은 Lombok 밑에 추가해주어야한다. (Lombok을 사용하기때문)

```java
@Mapper(componentModel = "Spring")
public interface UsedCarMapper {
    UsedCarMapper INSTANCE = Mappers.getMapper(UsedCarMapper.class);

    UsedCar toEntity(UsedCarRequestDto dto);
    UsedCarResponseDto toResponseDto (UsedCar entity);
}
```

인터페이스로 UsedCarMapper를 생성해준다.

@Mapper 어노테이션을 붙이고 해당 인터페이스를 통해 구현체를 생성한다는 것을 알린다.

이때 componentModel = "Spring" 은 스프링에서 컴포넌트로 등록하는 옵션이다.

첫 번째 필드는 interface가 매퍼클래스에 접근하게해주는 필드다.

### 실제 구현 클래스는 build 폴더아래 annotationprocessor 밑에 있다.

```java
package com.example.usedcar.mapper;

import com.example.usedcar.dto.requestdto.UsedCarRequestDto;
import com.example.usedcar.dto.responsedto.UsedCarResponseDto;
import com.example.usedcar.entity.UsedCar;
import javax.annotation.processing.Generated;
import org.springframework.stereotype.Component;

@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2023-05-19T13:34:20+0900",
    comments = "version: 1.5.3.Final, compiler: IncrementalProcessingEnvironment from gradle-language-java-7.6.1.jar, environment: Java 17.0.6 (Oracle Corporation)"
)
@Component
public class UsedCarMapperImpl implements UsedCarMapper {

    @Override
    public UsedCar toEntity(UsedCarRequestDto dto) {
        if ( dto == null ) {
            return null;
        }

        UsedCar.UsedCarBuilder usedCar = UsedCar.builder();

        usedCar.color( dto.getColor() );
        usedCar.mileage( dto.getMileage() );
        usedCar.price( dto.getPrice() );
        usedCar.name( dto.getName() );
        usedCar.model_year( dto.getModel_year() );

        return usedCar.build();
    }

    @Override
    public UsedCarResponseDto toResponseDto(UsedCar entity) {
        if ( entity == null ) {
            return null;
        }

        UsedCarResponseDto.UsedCarResponseDtoBuilder usedCarResponseDto = UsedCarResponseDto.builder();

        usedCarResponseDto.id( entity.getId() );
        usedCarResponseDto.color( entity.getColor() );
        usedCarResponseDto.mileage( entity.getMileage() );
        usedCarResponseDto.price( entity.getPrice() );
        usedCarResponseDto.name( entity.getName() );
        usedCarResponseDto.model_year( entity.getModel_year() );

        return usedCarResponseDto.build();
    }
}
```

## 여러 객체를 하나로 매핑하고 싶을 때

```java
public class PageDto {
	private Integer pageIndex;
	private Integer pageCount
}
public class MessageServiceDto {
  private String title;
	private String content;
	private String sender;
	private List<String> receiver;
	private String type;
	private Integer pageIdx;
	private Integer pageCnt;
}
public interface MessageMapper {
	MessageMapper INSTANCE = Mappers.getMapper(MessageMapper.class);

         //PageDto, RequestDto -> MessageServiceDto 매핑
	@Mapping(source="pageDto.pageIndex", target="pageIdx")
        @Mapping(source="pageDto.pageCount", target="pageCnt")
	MessageServiceDto toMessageServiceDto(PageDto pageDto, RequestDto requestDto);
}
```

PageDto와 MessageServiceDto를 MessageMapper를 통해 하나로 매핑할 수 있다.

### source는 매핑값의 자원을 명시하고 target은 매핑할 대상을 명시해준다.


## 매핑에 여러 파라미터가 들어갈 경우

```java
public class RequestDto {
        private String title;
        private String content;
        private String sender;
        private List<String> receiver;
        private LocalDateTime requestTime;
        private String type;
}
```

위 Dto를 아래와 같이 매핑하고 싶을 때 어떻게 해야할까?

```java
public class MessageListServiceDto {
	private String messageId;
	private Integer count;
	private String title;
	private String content;
	private String sender;
	private List<String> receiver;
	private LocalDateTime requestTime;
}
```

```java
MessageListServiceDto toMessageListServiceDto(String messageId, Integer count, RequestDto requestDto);
```
RequestDto에 없는 필드는 파라미터에 추가적으로 넘겨주면 된다.

