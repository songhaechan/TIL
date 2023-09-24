MVC-Pattern
=

스프링의 핵심 구조 MVC 패턴에대해 알아보자.

## Servlet이란?

Servlet은 사용자의 요청(request)를 받고 응답(response)를 내려주는 자바 프로그램이다.

이 Servlet은 스프링이 내장하고있는 톰캣이 생성해서 Servlet Container에서 관리한다.

아래와 같이 사용할 수 있다.

```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 애플리케이션 로직
    }
}
```

@WebServlet 어노테이션을 통해 **서블릿의 이름, url**을 매핑할 수 있다.

**HttpServlet을 상속**받고 **service**라는 메서드를 오버라이딩하여 사용한다.

이 서블릿은 사용자가 요청을 보내면 **서블릿컨테이너**에 등록되어 관리된다.

>init() 메서드로 서블릿을 컨테이너에 적재하고, destroy()를 통해 서블릿을 컨테이너에서 제거한다.(GC가 돌면 제거된다)

이래서 Tomcat이 서블릿 컨테이너라고 불리는 이유다.

### 서블릿 컨테이너가 주는 이점이 뭐지?

우선 서블릿 컨테이너가 웹서버와 소켓통신을 진행한다.

서블릿 하나하나 스레드를 하나씩 배분해서 멀티스레딩을 관리한다.
> 스레드를 무한정 생성하지는 않고 스레드풀에 맞게 생성해서 배분한다.

사용자의 Request와 서버의 Response는 많은 작업이 필요한 일이다.

우선 어떤 Request인지 어떤 url인지 어떤 Http method

Body에 어떤 데이터가 있는지 등등 분석(파싱)해야한다.

이 일을 개발자가 전부 해야한다면 **비즈니스로직**에 절대 집중할 수가 없다.

이 일을 서블릿이 받아서 전부 파싱하고 HttpServletRequest(or Response)에 담아서 사용할 수 있게된다.



### 서블릿 요청 흐름

<img src="https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2dffdd5e-e114-47e1-a28e-8a0e130b35d6%2FUntitled.png?table=block&id=ad85d212-4100-449a-ad4e-36830858e0d0&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2">



## 그럼 스프링은 어디서 움직이는 거야?

<img src="https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Ffba1fd42-d95d-463d-a32e-5a6d9dc92074%2FUntitled.png?table=block&id=82c579e8-8b92-4484-b071-00310640a99a&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2">

DI Container가 흔히 Bean을 관리하는 IoC컨테이너 혹은 스프링 컨테이너라고 불리는 공간이다.
>Bean에 대해서는 따로 정리하자.

간단히 IoC컨테이너는 @Component 가 붙은 클래스, 혹은 직접 @Configuration으로 생성한 Bean을 저장하는 그 공간이다.

여기서 중요한 점은 서블릿컨테이너가 빈에 접근하기위해선 스프링컨테이너를 거쳐야한다는 점이다.

## 서블릿과 컨트롤러 그리고 프론트 컨트롤러

```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 애플리케이션 로직
    }
}
```

맨 위의 서블릿 코드다.

우선 첫 번째로 클래스레벨에 url을 하나 매핑할 수 있다.

>수많은 클래스를 요청마다 생성해야한다.

두 번째로 메서드를 오버라이딩한다.

>컨트롤러는 사용 방식이 제각각이다. 메서드의 이름도 다양할 필요가 있을 것이고, HttpServletRequest(response)가 필요없는 컨트롤러도 있겠지만 현재로서는 모두 맞춰야한다.

### 그래서 Dispatcher Servlet[프론트 컨트롤러]이 등장했다.

Dispatcher Servlet도 이름 그대로 서블릿의 일종이지만 모든 컨트롤러의 수문장 역할을 한다.

<img src="https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F92ef2a5f-5d77-4ed3-9dab-418b67af8c20%2FUntitled.png?table=block&id=446db194-1038-4200-9674-b4dc90a50446&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2">

모든 요청을 프론트 컨트롤러가 맡아서 처리한다.

>기존에 서블릿으로만 작성하면 중복된 코드들이 매우 많았다. 하지만 프론트 컨트롤러가 이 중복된 코드들을 담당하고 다른 코드는 컨트롤러에 따로 작성한다.

여기서 더욱 자세히 알아보자.

<img src="https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fb2cee0d0-d963-4772-b113-bc52ee10627a%2FUntitled.png?table=block&id=25b7bc0c-0b8f-41b6-bfb3-14e4fdf4cb00&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=1700&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2">

첫 번째로 요청이 서블릿으로 들어오는 것은 지금껏 알아봤기때문에 이해하기 쉽다.

그리고는 **Handler Mapping**을 통해 **컨트롤러**를 찾는다.

> 이 과정으로 컨트롤러를 제각각 생성할 수 있는 것이다. 기존에는 service라는 메서드 틀에 맞춰야했지만 이젠 그럴 필요가 없어졌다.

제각각인 컨트롤러들을 구조에 맞게 변환시켜줄 필요가 있다. 이 일은 **Handler Adapter**가 수행한다.

이 후에 View 와 Model을 넘기면 View Resolver가 적절한 View를 찾아서 응답을한다.

> Json이라면 @ResponseBody로 바로 응답을 내린다. 뷰리졸버를 거치치않는다.