ArgumentResolver
=

ArgumentResolver는 요청이 들어온 파라미터중 필요한 데이터를 뽑아서 사용하기위한 기술이다.

### 아니 필요한 데이터만 뽑는다니? 애시당초 필요한 데이터만 줘야하는거 아니야?

맞다. 애시당초 필요한 데이터를 서버에 넘기는게 옳다.

하지만 암호화돼있는 데이터의 경우 이야기가 다르다.

서버까지는 완전히 암호화돼있는 상태로 도달해야한다.

>View단에서 복호화한다면 보안에 취약하다.

### 아니 그럼 Controller까지 받고 서비스단에서 처리하면 되는거아니야?

그렇게해도 되지만 코드가 **중복된다.**

암호화된 데이터를 복호화하는 과정이 자주 사용해야한다면 컨트롤러 곳곳에 작성해야한다.

하지만 ArgumentResolver는 파라미터에 커스텀어노테이션을 붙여줌으로써 ArgumentResolver가 동작하도록한다.

즉 컨트롤러에 복호화로직이 중북돼서 들어가지않게 해준다.

### 이번 프로젝트에서 Interceptor와 ArgumentResolver를 같이 사용한 이유

우선 프로젝트에서 필요한 것은 두가지 상황이였다.

1. 멤버의 세션을 단순히 검증만 수행

2. 멤버의 세션을 검증하고 추가로 세션에 있는 ID를 반환

사실 Interceptor에서 request에 memberId를 넣어서 반환할 수도 있다.

하지만 모든 요청에 memberId가 필요하진않았다.

굳이 필요없는 데이터를 request에 매번 넣어서 넘겨줄 필요가 없다고 생각한다.

> 저는 이렇게 생각하는데 이 이유가 맞을까요?

그래서 멘토님께서 ArgumentResolver를 사용하면 더 **깔끔**해질거라고 하신것 같다.

ArgumentResolver를 이용하면 필요한 컨트롤러에만 해당 memberId를 받아올 수 있다.

> ArgumentResolver는 단순히 파라미터로 넘어온 데이터만 가공하지않는다. request도 받아서 처리할 수 있다. http body까지도 처리가 가능하다.

---

마지막으로 요청 흐름이다.

### 1. Filter

스프링 바깥에 있다(서블릿 컨테이너) 스프링이 제공하는 예외처리에 해당하지않는다.

request나 response를 교체할 수 있다.

> 심지어 null도 넣을 수 있음...

보안에 관련된 사항을 주로 처리한다고하는데

사실 스프링바깥에서 보안처리를 할 일이 무엇인지 모르겠다.

### 2. DispatcherServlet

MVC 패턴에서 다뤘다.

컨트롤러들의 공통부분을 모아놓은 프론트 컨트롤러

URI를 분석해서 handlerMapping(Spring Bean)에 핸들러를 찾아오라고 요청한다.

찾아온 헨들러의 실행은 handlerAdapter에게 맡긴다.(결과만 받는다)

받은 결과를 Model과View라면 뷰리졸버에게 뷰이름을 넘긴다.

> 컨버터는 나중에 다루자!

---

Interceptor는 2번에서 실행된다.

핸들러매핑에게 요청하고 핸들러를 받는다고했지만

사실 HandlerExecutionChain을 반환받는다.

이 실행체인에 인터셉터가 등록돼있다면 loop을 돌며 인터셉터를 실행시키고 핸들러어댑터에게 실행을 요청한다.

---

Argument Resolver는 2번 속에서 핸들러어댑터에서 처리된다.

핸들러어댑터가 컨트롤러에게 실행을 요청하기전에

> Interceptor는 이미 처리된 상태

@RequestBody, @RequestParam등 파라미터 처리한다.

이때 우리가 생성한 Argument Resolver를 스프링에 등록했다면 실행되는 것이다.

그리고 이후 컨버팅과정도 어댑터가 처리한다.