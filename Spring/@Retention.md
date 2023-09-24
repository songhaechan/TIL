@Retention
=

ArgumentResolver를 구현하기위해선 커스텀어노테이션을 생성해야한다.

대충 이런식이다.

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface SessionAttribute {
}
```

@Target은 클래스레벨, 메서드레벨, 파라미터레벨로 설정할 수 있다. 직관적으로 다가온다.

그러던 중 @Retention에대해 궁금해졌다.

총 3가지 속성을 부여할 수 있다.

**RUNTIME**

**SOURCE**

**CLASS**

RUNTIME은 말 그대로 스프링이 올라오고나서 죽기전까지 계속 살아남는다.

@Autowired 나 @Component 같은 어노테이션이 RUNTIME 속성을 부여하는데

스프링이 올라오고나서 Reflection API를 통해 어노테이션 정보를 얻어와야한다.

---

SOURCE는 .java 파일까지 즉, 소스코드까지만 살아남는다.

롬복의 @Getter @Setter 어노테이션은 컴파일 후 .class파일(바이트코드)에선 사라진다.

즉 getter setter 메서드를 생성만하고 어노테이션정보는 사라진다.

---

CLASS는 .class 파일까지만 살아남는다.

.class 파일까지만 살아남는다는 말은 

클래스로더가 JVM에 클래스를 로드할 때 사라진다는 의미다.

JVM에 로드되면 사라지는 어노테이션은 왜 필요할까?

### 답은 jar파일(라이브러리)에 있다.

gradle이나 maven으로 주입한 라이브러리들 대부분은 jar파일로 이미 컴파일된 .class파일들이다.

SOURCE 속성과 RUNTIME 속성만 있다면 라이브러리들은 컴파일된 jar파일제공에 문제가 생긴다.

대표적으로 @NonNull 어노테이션이 그렇다.

> jar파일엔 소스가 없다. .class만 있다.