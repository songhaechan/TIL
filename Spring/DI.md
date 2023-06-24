DI컨테이너
=

```java
private final MemberRepository memberRepository = new MemoryMemberRepository();
```

위 코드는 인터페이스에 의존하고 있다.

그렇다면 DIP를 지킨걸까?

물론 DIP를 지킨것 처럼 보이지만, 사실은 지키고있지않다.

MemberRepository에'만' 의존해야지 MemoryMemberRepository라는 구현클래스에도 의존하고 있기때문이다.

중고차판매시스템에도 위와같은 코드가 수두룩하다.

해당 클래스는 다른 객체의 생성과 연결하는 책임을 가지며 자신의 로직까지 실행하는 책임을가진다.

SRP를 지키기위해선 관심사를 분리해야한다.

즉 사용영역(로직을 실행)과 구성영역(객체의 생성과 연결을 책임짐)으로 나누어 관심사를 분리해야한다.

```java
public class AppConfig {
    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }

    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(),discountPolicy());
    }

    public DiscountPolicy discountPolicy(){
        return new RateDiscountPolicy();
    }
}

```
위와같은 구성영역 (Assembler라고도 함)은 생성과 연결을 모두 담당한다.

사용영역을 보자.

```java
private final MemberRepository memberRepository;

    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
```

생성자를 통해 의존관계를 주입받고있다.

인터페이스만 알뿐 구체클래스에대한 어떠한 정보도 알지못한다.

캡슐화가 잘 이루어져있는 것이다.

```java
AppConfig appConfig = new AppConfig();
MemberService memberService = appConfig.memberService();
```

메인함수 안에서는 위처럼 AppConfig를 이용해 의존성을 주입시켜주면 된다.

이것이 바로 정확하게 DIP를 지켰다고 볼 수 있다.

OCP또한 잘 지킬 수 있다.

새로운 구체클래스를 생성하더라도 AppConfig의 객체생성만 변경하면 되기때문이다.

## 그렇다면 스프링의 DI컨테이너는 무엇일까?

IoC (제어의 역전)은 프레임워크가 개발자의 코드를 실행시키고 흐름을 프레임워크가 가져간다.

그래서 IoC컨테이너라고도 부른다.

하지만 요즘은 DI컨테이너로 통용해서 부른다.

DI컨테이너는 바로 위에서 AppConfig과 하는일이 동일하다.

```java
@Configuration
public class AppConfig {
    @Bean
    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(),discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy(){
        return new RateDiscountPolicy();
    }
}
```

@Configuration 어노테이션을 붙여줌으로써 해당 클래스가 설정클래스라는 것을 명시한다(프레임워크에게)

@Bean은 해당 메서드이름을 이름으로 저장된 객체는 반환하는 객체로하여 컨테이너에 저장하고 관리될 객체라는 뜻이다.

컨테이너는 해당 빈등록을 마치면 반환객체를 들고있게된다.

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
        OrderService orderService = applicationContext.getBean("orderService", OrderService.class);
```

ApplicationContext는 위에서 말한 스프링컨테이너이다.

해당 컨테이너를 생성할때 @Configuration을 붙인 클래스정보를 넘긴다.

그리고 해당 컨테이너에서 빈을 꺼내쓰는 (getBean) 방식이다.

오히려 컨테이너개념이 들어오고 코드가 복잡해진것같다.

이제부터 이 컨테이너가 왜 필요하고 무엇을 편리하게 만들어주는지 배워보자.