Filter vs Interceptor
=

게시판 프로젝트를 진행하며 로그인을 처리하는 부분은 컨트롤러 곳곳에 위치해있었다.

코드의 중복문제도 있지만 이후 유지보수에도 불리하다.

>인증 로직이 변경되면 코드 곳곳을 손봐야한다. 이는 SOLID원칙중 S (단일책임원칙)을 위반한다.

한 곳에서 모두 처리해야할 필요가 있다.

멘토님께서 추천해주신 방식은 Interceptor, AOP, Spring Security 가 있었고 오늘은 Interceptor와 Filter에 대해 알아보자.

## Filter

<img src="https://user-images.githubusercontent.com/8748075/86555900-d9095d00-bfa5-11ea-87f9-fac27fc6de3f.png">

Filter의 핵심은 스프링컨테이너에 있지않고 **서블릿 컨테이너에 있다는 점**이다.

사용자의 요청을 미리 받아서 처리하고, 응답을 받아서 마지막 처리를 돕는다.

데이터 압축, 인증, 암호화, 로깅 등을 수행할 때 사용한다.

## Interceptor

Interceptor는 디스패처 서블릿이 호출된 뒤 컨트롤러가 호출되기 전 호출된다.

즉 프론트 컨트롤러가 호출되고나서 Interceptor가 중간에 요청과 응답을 가로챈다.

### 아니 그런데 Filter랑 똑같은거 아닌가??

No

앞서 설명했듯 Filter의 핵심은 스프링컨테이너 밖에서 실행된다는 점이다.

즉 스프링의 예외처리가 이루어지지않는다. 스프링 밖에서 실행된다는 것이다.

> 하지만 Filter 객체는 빈으로는 등록이 된다...

```java
public class SessionInterceptor implements HandlerInterceptor {

    private final MemberService memberService;

    @Override
    public boolean preHandle(
            HttpServletRequest request,
            HttpServletResponse response,
            Object handler) {
        Long memberId = memberService.validateSession(request);
        request.setAttribute("memberId", memberId);
        return true;
    }
}
```

위 코드는 직접 HandlerInterceptor를 구현해서 preHandle 메서드를 오버라이딩했다.

이제 위 메서드는 지정한 컨트롤러가 호출되기전 무조건 실행된다.

공통 코드를 분리해냈다!!

>Filter를 사용하지않은 이유는 validateSession 메서드가 예외를 발생하기때문에 사용하지않았다.

### AOP와는 다르다.

얼핏보면 공통관심사와 부분관심사를 분리시키는 AOP와 비슷하기도하나 동작 방식이 완전히 다르다.

Interceptor는 요청과 응답을 가로채고 공통 코드를 작성한다.

AOP또한 공통 코드를 분리시키지만 빈으로 등록된 객체를 불러올 때 프록시 객체를 이용해 불러온다.

프록시를 사용하면 메서드를 좀더 세세하게 다룰 수 있다.

> AOP는 따로 정리하자. 깊다 내용이...