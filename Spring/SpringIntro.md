Spring Intro
==
스프링을 공부하며 간단한 **메모**를 남기려한다.

---

Spring 에서 API란
---
Spring에서 API는 객체를 그대로 내려줄 때 사용한다. 즉, JSON방식으로 객체데이터를 클라이언트에게 응답해주는것이다. 이때 @Responsebody 어노테이션을 쓰는데 HTTP메세지의 body부분에 객체를 직접 넣어주는 것을 말한다.

객체를 보내는데 어떻게 JSON으로 변환되어 내려가는지 궁금할 수 있는데, Spring 내부에서 JSONConverter가 작동하여 변환된다. 이 외에도 문자열을 그대로 내려주는 StringConverter도 존재한다.

---

TDD
---
지금까진 구현을 다 끝내고 TestCase를 작성하는 줄 알았지만, TDD란 테스트주도개발이라하는데, 먼저 TestCase를 작성하고 이에 맞는 구현체를 구현하는 것을 말한다. (PS 현업 백엔드 개발자에게 지금 뭐하고 있냐고 물어보면 60프로 이상은 테스트케이스 작성중이라고 대답한다고 한다. 그만큼 테스트케이스작성은 중요한것 같다...)

---

MVC 구조
----
옛날 옛적 호랑이 담배피우던 시절엔 Model과 View 그리고 Controller를 한 곳에 옹기종기 모아놓고 개발을 했다고한다.

개발방법이 진화함에따라 Model View Controller를 각각의 책임에 맞게 분리해냈고 이 개발패턴이 MVC패턴이다.

지금까지 이해한 내용은 Controller는 내장서버가 넘겨준 요청을 받아서 요청에 관한 데이터를 Model에 담아 return해주는데 이 때 viewResolver가 해당하는 View를 찾고 템플릿엔진을 연결시켜서 템플릿엔진이 Model을 이용해 렌더링한 후에 클라이언트에게 해당 응답을 내려준다.

---

컨테이너와 빈
----
후에 자세히 학습하겠지만 일단 빈이란 자바의 객체를 스프링이 생성하여 컨테이너에 보관한다는 점까지 이해했다. 구체적으로 어떤 구조로 생성하여 저장하는지는 모르겠다.

컴포넌트 스캔을 통해 (@Controller, @Service, @Repository, @Component 등) 스프링이 올라올때 빈으로 등록되고 해당 객체가 여러번 호출되더라도 단 하나의 인스턴스만 반환하게된다.

이를 싱글톤이라고 부른다.

빈을 이용해서 @Autowired 어노테이션을 생성자에 붙여주면 spring이 해당 객체를 주입해준다. 이를 DI(Dependency Ingection)[의존관계주입]이라 부른다.

---

DAO, VO, DTO
---
면접때 해당 개념에대한 답을 **하나도** 못했다.

열심히 공부해서 알아놓자

----
@Configuartion
---
스프링컨테이너에 빈으로 등록하는 방법은 @Controller 어노테이션같은 @Component 어노테이션을 붙여줘도 되지만, 다형성을 이용해서 유연한 프로그래밍을 위해 자바코드로 직접 빈을 등록해 줄 수 있다.

클래스레벨에 @Configuration을 붙여주면 해당 클래스는 자바 빈에 등록될 객체들을 모아놓은 클래스라는것을 알려주게된다.

```java
@Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository(){
        return new MemorryMemberRepository();
    }
}
```

위와 같이 메서드에 생성자로 객체를 생성해주면 이 인스턴스를 자바 빈에 등록해주게된다. @Bean에 추가로 값을 넘겨줘서 Bean id를 설정해주는 방법도 있다.

추가로 @Controller로 직접 빈으로 등록하고 @Autowired로 의존관계를 주입받을 수 있는데 이 외에도 필드 주입 setter주입이 있지만 보통 생성자로 주입받는것이 좋다.

필드 주입은 나중에 값을 변경해줄 수가 없고, setter는 public메서드로 선언해야하기때문에 외부에 노출되므로 좋지못하다.

