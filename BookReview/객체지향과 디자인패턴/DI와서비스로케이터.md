DI와 서비스로케이터
===
어플리케이션 영영과 메인영역
--
로버트 C 마틴에 따르면, 소프트웨어는 두 가지 영역으로 나눌 수 있다.


지금까지 알아본 SOLID을 이용해 객체지향적으로 객체들간의 관계를 정립함으로써 확장과 유지보수에 열려있는 설계에대해 학습했다.

이러한 핵심 로직을 수행하는 영역을 어플리케이션 영역이라 한다.

핵심적인 로직을 구현함에있어 SOLID원칙을 적용해야함을 명심하자.

어플리케이션영역에는 많은 객체들이 서로 의존하고있다. 이들 객체를 밖에서 생성하고 실행해줄 영역이 필요한데 이 영역이 메인영역이다.

그렇다면 메인영역은 어떤 방식으로 의존하는 객체들을 생성해줄까?

로케이터를 사용해 의존하는 객체들을 찾아줄 수 있다.

아래는 로케이터의 예시다.

```java
public class Locator{
    private static Locator instance;
    public static Locator getInstance(){
        return instance;
    }
    public static void init(Locator locator){
    this.instance = locator;
    }

    private JobQueue jobQueue; //비디오포맷변환 요청데이터
    private Transcoder transcoder; //비디오포맷변환객체
    public Locator(JobQueue jobQueue, Transcoder transcoder){
        this.jobQueue = jobQueue;
        this.transcoder = transcoder;
    }

    public JobQueue getJobQueue(){return jobQueue;}
    public Transcoder getTranscoder(){return transcoder;}
}
```

메인영역은 위 로케이터를 이용해서 의존하는 객체들을 찾아 locate해주게된다.(그래서 이름이 locator이다.)

위 클래스를 자세히보면 직접적으로 인스턴스를 생성하진않는다. 그저 의존하는 객체들을 담아놓는 그릇이라고 보면되겠다.

이제 메인영역이 위 로케이터를 어떻게 사용하는지 알아보자.

```java
public class Main{
    public static void main(String[] args){
        //JobQueue과 Transcoder는 상위모듈로 인터페이스
        //FileJobQueue와 FfmpegTranscoder는 하위 모듈 콘크리트클래스
        JobQueue jobQueue = new FileJobQueue();
        Transcoder transcoder = new FfmpegTranscoder();

        Locator locator = new Locator(jobQueue, transcoder);
        Locator.init(locator);
    }
}
```

메인영역에서는 필요한 객체를 생성해서 locator에 넘겨주게된다. (의존할 객체를 담는 그릇)

그 후에 Locator.init을 통해 locator인스턴스를 넘겨준다.

Locator마저 메인영역에서 생성해주는 것을 볼 수 있는데, 이는 의존의 방향과 관련이있다. 모든 의존성은 메인에서 애플리케이션영역으로 향한다.

이렇게 메인에서 필요한 객체를 교체하더라도 애플리케이션은 수정에 닫혀있게된다.

```java
JobQueue jobQueue = Locator.getInstance().getJobQueue();
Transcoder transcoder = Locator.getInstance().getTranscoder();
```

의존할 객체를 사용하는 입장에선(애플리케이션영역) 위와같이 Locator의 정적메서드를 이용해 locator인스턴스에 접근해 원하는 객체를 찾아서 연결시켜준다.

뒤에서 알아보겠지만 서비스로케이터는 단점이 존재하기때문에 DI(Dependenct Injection)을 사용하는 것이 일반적이다.

---

DI를 이용한 의존 객체 사용
--

DI를 한국말로 번역한다면 의존객체주입이라 부른다.

위에서 알아본 서비스로케이터는 일부 단점이 존재한다.(아직 알아보지 않았다.)

우선 DI에는 두 가지 종류가 있다. **생성자 방식**과 **메서드 방식**이 존재한다.

이펙티브자바의 첫 아이템에서 비슷한 주제의 내용을 다뤄본 경험이 있어서인지 두 방식의 장단점이 잘 이해됐다.

우선 생성자 방식부터 알아보자.


의존객체를 주입받는 객체
--
```java
public class Worker{
    private JobQueue jobQueue; // 인터페이스타입(상위타입)
    private Transcoder transcoder; // 인터페이스타입(상위타입)

    public Worker(JobQueue jobQueue, Transcoder transcoder){
        this.jobQueue = jobQueue;
        this.transcoder = transcoder;
    }
}
```
생성자를 설정할 때 의존하는 객체를 설정할 수 있도록 생성자 방식을 선택했다.

의존객체는 외부에서 생성해준다. 외부라함은 객체를 조립해줄 메인 영역을 뜻한다.



메인영역에서 생성
--
```java
final Worker worker = new Worker(jobQueue,transcoder);
```

서비스 로케이터 방식은 Locator 클래스에 담긴 객체를 선택적으로 넘겨받지만 DI방식은 위처럼 직접 객체를 넘겨받는다.

하지만 여기서 한 차례 더 객체지향적인 생각을 할 수 있다.

서비스로케이터와 DI 모두 메인영역에서 직접 객체를 생성하고 생성자에 넘겨주지만 객체를 한 번더 분리하면 더욱더 개방적인 설계가 가능하다.

Assembler '조립기'
---
```java
//메인 영역에서 분리된 클래스
public class Assembler{
    public void createAndWire(){
        JobQueue jobQueue = new FileJobQueue();
        Transcoder transcoder = new FfmpegTranscoder();
        this.worker = new Worker(jobQueue,transcoder);
    }
}
```

이렇게 분리해낸 '조립기'를 메인영역에서 생성해 위 메서드를 호출해주기만하면 후에 변경사항은 조립기에서만 발생하도록 할 수 있다.

스프링프레임워크에서 쓰이는 방식이 바로 위 방식이다.

이제 메서드 방식을 알아보자.

메서드방식
--

```java
public class Worker{
    private JobQueue jobQueue;
    public void setJobQueue(JobQueue jobQueue){
    this.jobQueue = jobQueue;
}
}
```

위와 같이 setter를 이용해 객체를 주입 받을 수 있다.

좋은 점은 무엇일까? 생성자는 이름이 없지만 setter메서드는 이름을 가질 수 있다. 내가 어떤 객체를 주입하는지 명확히 알 수 있다.

또 주입해야할 객체가 많아질 경우 헷갈리지않고 주입이 가능하다. 생성자는 주입할 객체가 많아지면 단순하게 넘기기 어렵다.

그리고 setter는 아래와 같이 메서드 체이닝방식으로 구현이 가능하다.

메서드 체이닝 방식
--
```java
public Worker setJobQueue(JobQueue jobQueue){
    this.jobQueue = jobQueue;
}
public Worker setTranscoder(Transcoder transcoder){
    this.transcoder = transcoder;
}

//사용하는 쪽 코드
worker.setJobQueue(instance1).setTranscoder(instance2)
```

하지만 단점도 존재하는데 메서드방식은 객체를 우선 생성하고 그 후에 객체를 주입해야하기때문에 객체가 미완성인 상태가 될 가능성이 있다.

생성자는 생성시점에 모든 객체가 조립되기때문에 미완성 객체가 될 수 없다.

책의 저자도 생성자방식이 안전한 방식으로 더 선호한다.

----

DI와 테스트
--
DI를 이용하면 테스트 또한 유연하게 할 수 있다.

앞에서 TDD방식에대해 간단히 살펴봤다.

TDD는 테스트코드를 먼저 작성하면서 변경이 일어날 부분을 자연스럽게 알아차릴 수 있다. 구현을 나눠서 하고 테스트도 나눠서하다보니 아직 구현이 끝나지않은 객체때문에 내 테스트가 보류중이라면 인터페이스를 활용해 Mock객체를 사용하게된다.

바로 이 부분이다. 자연스럽게 인터페이스를 도출하게되고 이 부분은 더욱더 객체지향적인 설계로 유도한다.

그렇다면 DI와 테스트의 관계는 무엇일까?

생성자 방식의 DI의 가장 큰 이점이 무엇이었는지를 다시 생각해보면 의존하는 객체가 변경되었을 때 조립하는 쪽에서 객체를 말 그대로 바꿔주기만하면 사용하는 쪽이나 다른 곳에서는 수정할 필요가 전혀 없다.

테스트에서도 마찬가지로 적용되는 장점이다. Mock객체를 이용해야하는 상황이라면 다른 코드는 손대지않고 Mock객체를 생성해 바꿔주기만하면 된다.

**질문** : DI를 사용하지 않아도 테스트에 영향을 주지않는 방법이 있지않을까?

---

서비스로케이터를 이용한 의존 객체 사용
--

책의 저자도 서비스로케이터의 문제점으로 DI사용을 권하고있지만, 간단하게 단점을 알아보도록하자.

서비스 로케이터는 메인영역에서 의존하는 객체를 생성한 뒤에 로케이터의 생성자에 할당한다. 그 뒤에 생성한 로케이터인스턴스를 로케이터 내부에 할당한다.

```java
FileJobQueue jobQueue = new FileJobQueue();
FfmpegTranscoder transcoder = new FfmepegTranscoder();
ServiceLocator locator - new ServiceLocator(jobQueue,transcoder);
ServiceLocator.load(locator);
```

로케이터를 사용해 의존객체를 전달 받을 객체는 로케이터안에 정의돼있는 static 메서드를 이용해 로케이터 인스턴스를 가져와 의존객체를 할당 받는다.

이 때 문제점은 무엇일까?

의존역전원칙 위반
--
첫 째로 로케이터의 생성자는 public 이기때문에 고수준에서 저수준으로 접근이 가능하다는 점이다.

```java
class Worker{
    //고수준모듈에서 저수준모듈 직접접근 (인터페이스를 통한 접근이 아님)
    ServiceLocator newLocator = new ServiceLocator(new DbJobQueue, oldLocator.getTranscoder());
    //DbJobQueue (저수준모듈)을 직접 사용하고있음
}
```
인터페이스 분리 원칙 위반
--
두 번째는 로케이터를 재사용함에있어 어떤 객체는 필요없는 의존객체를 담고있는 로케이터를 사용할 가능성이 있다(필요없는 의존 발생). 인터페이스 분리원칙을 적용해 분리해내더라도 (혹은 단일책임원칙) 코드의 중복이 발생한다. 

로케이터에는 JobQueue와 Transcoder 의 콘크리트 클래스를 담고있는데 Worker클래스는 JobQueue만 필요하다 가정할 때, 필요없는 Transcoder를 의존하고있다.

인터페이스 분리 원칙을 위반하고 있는셈이다.

인터페이스를 분리해낸다 하더라도 서로 다른 인터페이스가 Jobqueue를 구현하는 것은 변함없기때문에 코드의 중복도 발생한다.

의존성제거가 목적이 아닌 분리자체가 목적인 인터페이스분리는 코드의 중복만 증가시킨다.

여러 객체 전달 불가능
--
Locator는 의존하는 객체를 private으로 싱글턴을 (Locator가 하나라는 가정하에)보장하고있다. 

하지만 다른 종류의 인스턴스가 필요하면 새로운 로케이터를 생성해서 전달해야한다. 메서드의 이름을 붙이기에도 쉽지않아진다.

```java
getJobQueue1();
getJobQueue2();
// 메서드 이름을 짓기 좋지않다.
```

결론 : DI를 사용하자. DI를 지원하지않는 프레임워크를 사용한다면 서비스로케이터를 사용하자. (스프링은 DI를 지원함으로 DI사용하자.)

