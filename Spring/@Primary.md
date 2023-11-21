## @Primary annotation

@Primary는 S3 Configuration을 진행하던 중에 만난 어노테이션이 였다.

```java
    @Bean
    @Primary
    public AmazonS3 amazonS3Client() {
        AWSCredentials credentials = new BasicAWSCredentials(accessKey, secretKey);

        return AmazonS3ClientBuilder
                .standard()
                .withCredentials(new AWSStaticCredentialsProvider(credentials))
                .withRegion(region)
                .build();
    }
```

S3를 사용자(나)를 생성해 빈으로 등록시키는 코드다.

## 우선 @Qualifier 부터 알아보자

https://www.baeldung.com/spring-autowire 

밸덩을 참조했다.

아래와 같은 빈 두 개가 있다.

```java
@Component("fooFormatter")
public class FooFormatter implements Formatter {
    public String format() {
        return "foo";
    }
}

@Component("barFormatter")
public class BarFormatter implements Formatter {
    public String format() {
        return "bar";
    }
}
```

Formatter 인터페이스의 구현체 FooFormatter, BarFormatter 두 개가 있고, @Component로 빈으로 등록된 상태다.

여기서 빈을 주입받는 코드를 보자.

```java
public class FooService {
    @Autowired
    private Formatter formatter;
}
```

이렇게 Formatter를 주입받으려하면 NoUniqueBeanDefinitionException 예외가 발생한다.

주입하려는 객체가 타입이 같고 고유하지 않다는 것이다.

이때 @Qulifier를 사용한다.

```java
public class FooService {
    @Autowired
    @Qualifier("fooFormatter")
    private Formatter formatter;
}
```

구현체 이름을 명확하게 적어주는 용도로 사용한다.

## 그럼 @Primary란?

https://www.baeldung.com/spring-primary

마찬가지로 밸덩을 참고했다.

```java
@Configuration
public class Config {

    @Bean
    public Employee JohnEmployee() {
        return new Employee("John");
    }

    @Bean
    public Employee TonyEmployee() {
        return new Employee("Tony");
    }
}
```

@Qulifier 예시와 비슷한 예시다.

Bean으로 등록하기는 하는데, 타입이 같다. 당연히 그냥 주입받으면 NoUniqueBeanDefinitionException가 터진다.

    We apply it at the injection point along with @Autowired. In our case, we select the beans at the configuration phase so @Qualifier can’t be applied here. <Baeldung - Primary>

인젝션 포인트, 즉 **주입받는 쪽**에서 @Autowired와 @Qualifier를 사용해 구체타입을 명시해줬다.

그런데 이건 순전히 사용하는 쪽이고, Configuration phase, 설정 측면에서는 사용할 수가 없다.

이때 설정 파일에서(@Configuration이 붙은 클래스에서) 사용하는 어노테이션이 @Primary다.

```java
@Configuration
public class Config {

    @Bean
    public Employee JohnEmployee() {
        return new Employee("John");
    }

    @Bean
    @Primary
    public Employee TonyEmployee() {
        return new Employee("Tony");
    }
}
```

위 예시 코드에 @Primary만 TonyEmployee에 붙어다.

이렇게 되면 특정 빈이 **우선순위를** 갖고 주입된다.

즉, 스프링 컨테이너에게 빈의 우선순위를 알려주는 어노테이션이라 생각할 수 있다.

추가로! @Bean과 같이 쓰기도 하지만 @Component와 같이 사용되기도 한다.

## @Bean, @Component 무슨 차이일까?

일단 처음 언급한 S3 사용자를 빈으로 등록하는 전체 코드를 보자.

```java
@Configuration
public class S3Config {
    @Value("${cloud.aws.credentials.access-key}")
    private String accessKey;

    @Value("${cloud.aws.credentials.secret-key}")
    private String secretKey;

    @Value("${cloud.aws.region.static}")
    private String region;

    @Bean
    @Primary
    public AmazonS3 amazonS3Client() {
        AWSCredentials credentials = new BasicAWSCredentials(accessKey, secretKey);
        return AmazonS3ClientBuilder
                .standard()
                .withCredentials(new AWSStaticCredentialsProvider(credentials))
                .withRegion(region)
                .build();
    }
}
```

난 AmazonS3 객체를 나의 정보를 이용해 생성해서 빈으로 등록하고 싶다!

그런데 AmazonS3는 내가 직접 만든 클래스가 아니다.

Gradle에 AmazonS3 의존성을 주입하고 얻는 외부 클래스다.

이 경우에 @Bean 어노테이션을 사용한다.

위 예시처럼 메서드 레벨에 붙어서 작동한다.

반대로 @Component는 사용자가 직접 정의한 클래스레벨에 붙여서 사용한다.

다시말해 @Primary는 클래스레벨에 @Component와 같이 붙을 수도 있고, @Bean과 함께 메서드레벨에도 붙을 수 있다.

또한 @Bean은 수동으로 객체를 등록할 때 사용하고(@Configuration과 함께) @Component는 스프링이 올라오면서 컴포넌트 스캔을 시작하면서 빈으로 등록된다.

## 왜 @Bean은 @Configuration과 함께 사용할까?

@Configuration을 빼고 @Bean으로 등록하면 등록이 안될까?

일단 결론은 빈으로 등록이 된다.

하지만 **싱글톤이 깨지게 된다.**

@Configuration 어노테이션이 붙어야지만 싱글톤을 보장해준다.

스프링은 프록시객체 패턴을 사용해 객체가 단 한 번만 생성될 것을 보장한다.

## 너무너무 돌아왔다 그래서 결론

지금 S3 사용자 객체를 생성하는 코드에

@Primary는 아무 짝에도 쓸모가 없는 어노테이션이다.

## @Primary와 @Qualifier 활용

지금 나에게 쓸모가 없는 어노테이션이지만... 

이 두 어노테이션은 함께 짬뽕해서 구성하면 나름 쓸모가 있다.

Default로 주입받을 빈은 @Primary로 등록한다.

그리고 부차적으로 자주 사용하지않는 빈은 @Qualifier로 사용할 수 있다.


