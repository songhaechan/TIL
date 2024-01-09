## Optional을 메서드 파라미터에 전달하는 것에 대해

https://dzone.com/articles/using-optional-correctly-is-not-optional

위 문서의 item 16을 읽고 정리한다.

일단 첫 줄에서 부터 파라미터에 전달하면 안되는 이유가 나오는데

>Don't force call sites to create Optionals.

호출 부에서 Optional을 만들도록 강제하지 말라는 것이다.

그 이유로는 크게 코드의 복잡성 증가, 의존성 증가, 고비용 이다.

### 코드의 복잡성 증가?

이 부분은 사실 납득이 잘 가지 않았다.

Optional을 사용하면 적어도 코드가 간단해진다고 생각했는데...

null체크 코드가 더 복잡하다고 생각한다.

### 의존성 증가와 고비용

이 부분은 공감한다.

Optional을 메서드 파라미터로 사용하게되면 이 메서드를 사용하는 모든 곳은 Optional을 넘겨야한다.

null이 아님이 확실한 객체 조차도 모두 Optional을 사용해야한다.

이 글에서 언급하길 일반적인 객체보다 메모리를 4배 소모한다고 한다.

## Optional Method Parameter Check

위 글에 이어서 무조건 Optional을 사용하지 말라는 것이 아니라 아래 글을 읽고 판단해보라하여

아래 글도 읽어봤다.

https://dzone.com/articles/optional-method-parameters

## 왜 null을 그냥 전달하면 안되는가?

null을 던졌을 때 해당 메서드가 null을 지원하는지를 파악하기위해 코드 끝까지 다 뒤져봐야한다는 비용이 있기때문에 null을 던져서는 안된다.

애초에 null을 지원하지 않아야 위의 쓸 때 없는 작업을 하지 않는다는 것이다.

만일 null을 지원하는 메서드가 있다 하더라도, 해당 메서드는 null을 지원하지않는 메서드로의 수정은 하지 못한다.

즉 제약이 생겨버리게된다.

호출자던 수신자던 둘 모두에게 좋은 점이 없다.

NPE 문제는 덤이다.

## Optional을 메서드 파라미터에 전달하면 안되는 또 하나의 주장

StackOverflow의 답변 내용이다.

```java
public void myMethod(Optional<String> optionalArgument) {
    // some code
    if (optionalArgument.isPresent()) {
        doSomething(optionalArgument.get());
    } else {
        doSomethingElse();
    }
    // some code
}
```

>Using Optional parameters causing conditional logic inside the methods is literally contra-productive.

논리적 분기문이 생겨난다는 스택오버플로우의 주장인데, 동의한다.

그런데 이 글의 작성자는 반박한다.

```java
public void myMethod(Optional<String> optionalArgument) {
    // some code
    doSomething(optionalArgument);
    // some code
}
```

'아니 그냥 넘길 수도 있잖아?'

그리고 Optional을 사용하지않는다면 스택오버플로우 답변대로면 코드는 더 복잡해진다.

```java
public void myClient() {
    // some code
    String optionalArgument = ...
    if (optionalArgument == null) {
        myMethod();
    } else {
        myMethod(optionalArgument);
    }
    // some code
}
```

유일한 대안은 오버로딩을 통한 해결이라는 것이다.

즉, 호출하는 쪽을 보고 결정해야한다고 이 글의 작성자는 말한다.

호출하는 쪽이 잠재적으로 null인 객체를 넘길 수 있다면 오버로딩을 통해 해결할 수 있고 (코드는 복잡해 지지만)

그게 아니라면 Optional을 사용하는 것도 괜찮다고 말한다.

## Function Does More Than One Thing

그리고 위 논리적 분기의 주장에서 함수가 1가지 일 이상의 일을 하니 문제가 있다 라고하는데

```java
public void myMethod(Optional<String> optionalArgument) {
    // some code
    String argument = optionalArgument.orElse("reasonable default");
    doSomething(argument);
    // some code
}
```

isPresent() 를 통한 코드 작성이 아닌 위와 같이 작성한다면 그다지 나쁘지 않다는 주장을 한다.

## 내 생각

두 글을 읽고 든 생각은 호출하는 쪽 그리고 고비용이다.

Optional이 기존 참조보다 4배 더 비싸다는 것과 의존성이 증가한다는 것을 명심해야하고, 정말 필요한 경우가 아니라면 되도록 피하는 것이 좋을 것 같다.

두 번째 글의 논리적 분기에대한 주장은 사실상 무의미하다고 생각한다.

orElse 문으로 최대한 회피가 가능하고, Optional을 사용하지 않아도 분기가 생긴다는 길 수 있다.(Optional은 무조건적인 분기가 생기긴하지만...)