API(CRUD) 설계
=
## 구조
### Controller - Service - Repository(JPA)

Controller 는 무엇일까?

사실 서블릿, 톰캣, WAS,타임리프, 디스패쳐서블릿 등등... 너무 많은 개념이 들어있어 혼돈스럽지만 우선은 대략적인 흐름을 보고자한다. 후에 더 자세한 원리를 정리해야겠다.

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbED6o9%2Fbtrx1wyKwpF%2FNtSlrTohpAI79l6MA95SZ1%2Fimg.png)

위 사진은 일반적으로 @Controller 를 붙여주면 view를 반환하는데 이 과정을 뜻한다.

1. Dispatcher Servlet은 프론트 컨트롤러 라고도 불린다. client의 모든 요청을 앞단에서 받아 처리한다.

2. Handler Adapter는 요청에 알맞은 컨트롤러를 찾아 컨트롤러에게 위임한다. (Post인지 Get인지 Delete인지 매핑된 정보에 의해서)

드디어 컨트롤러가 나왔다.

3. 컨트롤러는 요청을 위임받고 서비스단에서 로직을 처리 후 View Name을 반환한다.

4. Handler Adapter는 View Name을 Dispatcher Servlet에 반환하고 다시 View Resolver에게 해당 View를 찾아달라고 한다.

이렇게 View를 반환받고 클라이언트에게 반환하는 방식이다.

### 그럼 View가 아닌 Data를 반환한다면?

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb3McJC%2Fbtrx1IGcnGs%2F2iHFmw3bbqasfCJzwCKYuK%2Fimg.png)

Dispatcher Servlet이 요청을 제일 처음 받고 Handler Adapter를 통해 Controller에 요청을 위임하는 것은 같다.

Controller는 Service단에서 요청을 처리하고 객체를 반환하는데 이 때 ResponseEntity에 감싸서 반환한다.

View를 반환할 때와는 다르게 ViewResolver는 개입하지않고 HttpMessageConverter가 작동한다.

4번에서 메세지를 객체로 6번에서 객체를 메세지로 변환하는데 작동한다.

여기서 메세지라함은 요즘 주로사용하는 JSON이 될 수 있다.

```java
@GetMapping(value = "/users")
    public @ResponseBody ResponseEntity<User> findUser(@RequestParam("userName") String userName){
        return ResponseEntity.ok(userService.findUser(user));
    }
```

여기서 User 객체를 ResponseEntity로 감싸서 보내는 것을 확인할 수 있고 userService.findUser(user)를 통해 컨트롤러가 서비스단에서 요청을 처리함을 확인할 수 있다.

여기서 중요한 것이 @ResponseBody인데 이 어노테이션이 객체를 JSON으로 반환해 HTTP body에 넣겠다는 의미이다.

하지만 주로 @RestController 어노테이션을 사용해 View를 반환하는 컨트롤러와 Data를 반환하는 컨트롤러를 따로 작성한다.

@RestController = @Controller + @ResponseBody  인 셈이다.

### 여기서 WAS와 WS를 간단히 짚고 넘어가자.(Dispatcher Servlet을 위해...)

WS(Web Server)는 정적인 데이터를 처리한다. 단순한 html, jpg 등등을 가공하지않고 그대로 반환한다.

WAS는 동적인 처리를 담당한다. View를 반환하거나 데이터를 반환할때 로직과 DB를 이용해 요청을 처리하는 과정이 필요하기때문에 이 과정은 WAS가 담당한다.

자 그렇다면 톰캣은 무엇일까?

WS와 WAS의 기능을 합쳐놓은 것이라 생각하면된다.

톰캣은 WAS이지만 WS를 내장하고있다.

위에서 Dispatcher Servlet이 가장 먼저 요청을 받는 다고했지만 좀 더 자세히 말하자면 WS가 가장 먼저 요청을 받는다.

정적인 요청인지 동적인 요청인지에따라 정적이라면 WAS를 거지치지않고 직접처리하고 동적인 요청이라면 WAS에 요청을 위임한다.

굳이 정적인 데이터를 WAS까지 개입할 이유가 없기때문이다(느리다).

이때 Dispatcher Servlet이 바로 WAS안에 있기때문에 요청을 제일먼저 받는다고 설명한 것이다.

즉 **동적인 요청**을 제일 먼저 받는 서블릿이다.

### ViewResolver 란?

thymeleaf를 이용하며 뷰리졸버라는 존재를 알았다. 타임리프는 동적인 웹페이지를 생성하는데 도움을 준다.

말 그대로 해당하는 View를 찾아서 반환하는 역할을 한다.

굳이 소속을 따지자면 스프링프레임워크에 속해있다.

---

```java
@GetMapping("/{id}")
    public ResponseEntity<UsedCarResponseDto> getUsedCar(@PathVariable Long id) {
        return ResponseEntity.ok(usedCarService.getUsedCar(id));
    }
```

여기서 ResponseEntity가 뜻하는 바를 간단히 알아보면 사용자의 요청에 컨트롤러는 응답 메세지를 HTTP방식으로 보내야한다.

이때 헤더와 바디를 작성해서 보내야하는데 위에서 설명했듯이 바디에는 전달할 데이터를 담는다. 

ResponseEntity.ok()는 응답코드 200이라는 성공 메세지를 담은 헤더를 만들고 바디를 usedCarService.getUsedCar(id) 로 작성한다.

```java
@PostMapping
    public ResponseEntity<Long> saveArticle(@RequestBody UsedCarRequestDto usedCar) {
        return ResponseEntity.ok(usedCarService.saveUsedCar(usedCar));
    }
```

위에서 @RequestBody는 HTTP메서드의 본문에 있는 데이터를 객체로 변환해준다.







