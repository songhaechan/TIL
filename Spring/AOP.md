AOP
=

AOP(Aspect Oriented Programing)은 관점지향 프로그래밍으로 스프링에서 제공하는 프록시기능이다.

AOP에서의 관점이란 핵심적 관점과 부가적 관점으로 나뉜다.

현재 프로젝트에서 예시를 들어보자.

```java
// 게시글 생성
    @PostMapping
    public CommonIdResponse saveBoard(
            HttpServletRequest request,
            @Valid @RequestBody CreateBoardRequest createBoard) {
        sessionValidation(request);
        Long memberId = (Long) request.getAttribute("memberId");
        return boardService.createBoard(memberId, createBoard);
    }
```

게시글을 조회하는데 sessionValidation()이라는 메서드가 들어있다.

이 메서드의 핵심적 관점은 **게시글을 생성**하는 일이다.

사용자의 세션을 검증하는 일은 이 관점으론 **부가적 관점**에 해당한다.

AOP는 이러한 부가적 관점으로 보여지는 부분을 모두 한 곳에서 관리/실행 되도록 도와준다.

## AOP의 용어

AOP엔 몇 가지 용어들이 나오는데... 큰 흐름을 기억한다면 용어들은 이해하기 쉽다.

>그래도 어렵다.

### Aspect (관점)

공통 적용할 기능을 의미한다. 부가적 기능을 정의한 Advice와 Advice의 적용위치를 결정하는 Point Cut의 조합으로 이루어진다.

> 여기서 부가적 기능을 세션검증이라 생각하고(Advice) 컨트롤러 중에서 세션을 검증할 메서드의 위치 (Point Cut, Join point)
>
> 사실 Join Point가 적용할 메서드의 위치를 나타내고 Point Cut이 어떤 조인포인트를 사용할 지 결정한다.

### Proxy

AOP가 동작할 수 있는 이유다.

BoardService를 빈에 등록하고 이 BoardService의 메서드들이 조인포인트로서 어드바이스에서 실행될 때 **가짜 객체**를 스프링이 생성한다.

### Target

실제 비즈니스 로직을 실행하는 객체

### Introduction

타겟에는 없는 새로운 메서드나 필드를 추가할 때 사용하는 객체


### 그래서 어떻게 사용하지?

클래스레벨에 @Aspect를 적용하면 AOP기능을 하는 객체라고 스프링에게 선언한다.

Advice는 @After @Before @Around 등 5가지가 있다. Around가 주로 많이 사용된다.

> @Around = Traget 메서드 호출 이전과 이후 모두 적용

보통 excution을 사용해 타겟팅할 메서드나 클래스를 선언한다.