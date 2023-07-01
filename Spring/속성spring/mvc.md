# Sl4j [로깅]

클래스레벨에 롬복으로 @Sl4j 넣어주면 사용가능

로그레벨은 개발은 debug, 운영은 info로 !!

차후에 더 공부하자

# 요청 매핑

## @RestController = @Controller + @ResponseBody

@ResponseBody는 HTTP메세지 바디에 직접 데이터를 전달한다. (JSON 응답할때 매우매우 많이 사용)

@ResponseBody가 없으면 일반 @Controller는 반환값이 String일때 뷰이름으로 인식해서 뷰리졸버가 동작함.

API통신할땐 RestController를 사용하자.

---
## @RequestMapping("/hello-basic")

이 방식은 GET PUT POST PATCH DELETE 등 모든 메서드를 허용하기에 좋지않음.

@GetMapping 같은 구체적인 어노테이션을 이용해서 url을 매핑하자.

@RequestMapping은 클래스레벨에 붙여서 상위루트를 표현할때 사용하자!!

---
## @PathVariable

```java
@GetMapping("/mapping/{userId}")
  public String mappingPath(@PathVariable("userId") String data) {
      log.info("mappingPath userId={}", data);
      return "ok";
  }
```

{userId}는 뭐다? 경로변수다.

그래서 받을땐 @PathVariable로 받아주자.

추가적으로 파라미터명과 경로변수명이 같으면 ("userId")생략 가능하다.

@PathVariable은 HTTP api 통신할때도 사용 많이한다.

---

강의 내용중에 헤더정보에따라 매핑을 시켜주는 내용이 있다. MVC1편을 참고 (MVC1편 기본기능)

---

# 요청 매핑 (API)

## API url 매핑

목록 조회 GET /users (경로변수받을 필요 없다.)
회원 등록 POST /users (PRG패턴 꼭 기억)
GET /users/{userId} (단건조회는 id받아야해)
PATCH /users/{userId}
DELETE /users/{userId}

```java
@GetMapping("/{userId}")
    public String findUser(@PathVariable String userId) {
        return "get userId=" + userId;
    }
```
---

헤더를 조회하는 방법이 나와있는데 필요하면 참고(MVC1편 기본기능)

# HTTP 요청 데이터 조회

기억하자 총 3가지 안에서 지지고볶는다.

1. GET 쿼리 파라미터 ?key=value

2. POST HTML FORM 메세지바디에 쿼리파라미터형식

3. HTTP message body에 JSON등을 직접 전달

1,2는 받을때 비슷하지만 3은 다르다.

```java
@RequestMapping("/request-param-v1")
      public void requestParamV1(HttpServletRequest request, HttpServletResponse
  response) throws IOException {
          String username = request.getParameter("username");
          int age = Integer.parseInt(request.getParameter("age"));
          log.info("username={}, age={}", username, age);
          response.getWriter().write("ok");
      }
```

request로 값을 꺼내서 사용이 가능하다.

이 방법은 1번과 2번 모두 동일하게 값을 꺼낸다.

하지만 이렇게 하면 귀찮으니 어노테이션을 쓰자 ^^(위는 서블릿 방식임)

---
## @RequestParam

```java
@ResponseBody
  @RequestMapping("/request-param-v3")
  public String requestParamV3(
          @RequestParam String username,
          @RequestParam int age) {
      log.info("username={}, age={}", username, age);
      return "ok";
}
```

@RequestParam은 요청에서 그대로 값을 꺼내서 담아주지만 주의할점은 파라미터명이 바인딩될 변수명이랑 같아야함.

RequestParam은 생략할 수 있다고하지만 하지말자 명시적으로 밝히자.

**필수적으로 받을지 말지도 결정이가능하다 (옵션을 검색)**

**디폴트로 받을값도 결정이 가능하다**

**파라미터를 Map으로도 조회가능하다**

쓰고싶다면 검색을 하자.

---

## @ModelAttribute

사실 보통 요청으로 값을 받으면 대부분 객체로 변환해서 지지고볶을때가 많다.(아닐때도 있음)

그럴때 @ModelAttribute !!


```java
@ResponseBody
    @RequestMapping("/model-attribute-v2")
    public String modelAttributeV2(HelloData helloData) {
        log.info("username={}, age={}", helloData.getUsername(),
    helloData.getAge());
        return "ok";
    }
```

보통 이렇게 @ModelAtrribute를 생략해서 쓴다.

HelloData 객체에 setter를 찾아서 바인딩한다.

### 절대로 컨트롤러에 위에처럼 엔티티를 노출하지말고 dto를 사용하자 

### 추가로 엔티티에 setter 남발하지말고 메서드를 따로 빼자

### 꼭 기억해야할 것은 @RequestParam, @ModelAttribute는 메세지바디에 직접 데이터가 넘어오면 못쓴다. 즉 1,2번 경우에만 사용가능하다.

---

## 아니 그럼 바디에 직접 넘어오면 ?? @RequestBody

```java
@ResponseBody
    @PostMapping("/request-body-string-v4")
    public String requestBodyStringV4(@RequestBody String messageBody) {
        log.info("messageBody={}", messageBody);
        return "ok";
    }
```

이번 프로젝트에서 가장 많이 사용할 RequestBody!!

사실 위에선 데이터를 꺼내서 바인딩해야하는데 그럴 필요가 없다.

```java
@ResponseBody
  @PostMapping("/request-body-json-v3")
  public String requestBodyJsonV3(@RequestBody HelloData data) {
         log.info("username={}, age={}", data.getUsername(), data.getAge());
        return "ok";
  }
```

바로 객체를 지정하면 JSON컨버터가 작동해서 자바객체로 변환해준다.

---

## 이쯤에서 승진이의 코드를 보자.

```java
@PostMapping
    public ResponseEntity<IdResponse<Long>> saveArticle(@RequestBody ArticleRequestDto article) {
        return ResponseEntity.ok(articleService.saveArticle(article));
    }
```

상위엔 RestController가 붙어있어서 바디에 직접 입력이 가능하다.

Post매핑으로 요청을 처리한다.

ResponseEntity를 사용한다. (헤더나 상태코드를 자유롭게 넣을 수 있다.)

ok메서드를 사용해 응답데이터와 함께 내려준다.

@RequestBody를 사용해 자바객체로 바로 변환한다.

---

메세지 컨버터 공부하자.







