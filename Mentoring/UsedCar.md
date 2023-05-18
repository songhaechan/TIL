정규형
=
### 데이터베이스를 설계할 때 관계의 이상현상을 방지하기위해 관계를 분리하는 규칙이다.

## 제 1 정규형

제 1 정규화란 하나의 컬럼은 원자값을 가져야한다는 것이다.

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FNRCAQ%2Fbtrj110vGrs%2Fn1XpWYrc5RdtwYRpeuwHQK%2Fimg.png)

과목 컬럼은 하나의 필드에 두 가지 데이터가 들어가있다.

원자값을 가지지못하기때문에 테이블을 분리해야한다.

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FVJ4EU%2Fbtrj2ABojKV%2F9t35vgqac4GMBVYYBIIKs0%2Fimg.png)

## 제 2 정규형

제 2 정규화는 제 1 정규화를 만족시켜야하고, 모든 컬럼이 **완전 함수 종속**을 만족시켜야한다.

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FpwqKp%2Fbtrj2WEouSq%2FrHLj2INEMyM1PkzYkWATK1%2Fimg.png)

위 테이블은 학생번호와 과목이 복합키이다.

학생번호와 과목을 알아야 성적을 알 수 있기때문이다.

그런데 특정과목의 지도교수는 과목만알면 지도교수를 알 수 있기때문에 학생번호에는 종속적이지못하므로 부분종속이라 할 수 있다.

이런 부분종속을 없애야하는 것이 제 2 정규화이다.

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FskXwR%2Fbtrj0U2B6V9%2Fs4xUYsd8DBZwLLew4gJ0Ik%2Fimg.png)

위와 같이 테이블을 분리하면 지도교수는 과목이라는 키에 완전종속을 만족한다.

## 제 3 정규형

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FxUsSs%2Fbtrj4sJJchb%2FjZhQDFOYYSNkqM75cG87w0%2Fimg.png)

위 테이블은 ID를 알면 등급을 알 수 있고 등급을 알면 할인율을 알 수 있다.

A->B->C 라는 추이성을 보이고있는데 제3정규형은 이 추이성을 분리해야한다.

A->B B->C 로 분리한 결과는 아래와같다.

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdYnxPO%2FbtrP8O3VPDg%2FDfyMkh8K5mnKFBp35BOJYk%2Fimg.png)


Test Double
=

책에서 Mock객체에대해 읽은적이 있다. 이 Mock객체도 Test Double의 한 종류이다.

테스트를 해야하지만 테스트하기 모호한 경우가있다. 예시를 보면 더 쉽게 이해할 수 있다.

## Dummy

단순히 인스턴스화된 객체가 필요할 때 사용한다. 객체의 기능은 사용하지않고 동작을 보장하지않는다.

예를 들자면 인터페이스를 상속받고 구현은 빈구현을 통해 객체만 생성하고 동작하지않는 객체를 말한다.

## Fake

Dummy와는 다르게 아예 비어있지않다. 내부 로직이 너무 복잡할때 중간 과정은 건너뛰고 마지막 로직만 구현한 단순화된 객체를 말한다. 동작은 하지만 정교하지 못하다.

그래서 실질적인 객체로 사용될순 없지만 테스트시에 원하는 결과를 보기에 적합하다.

## Stub

TDD를 읽으며 초록막대를 보기위해 빠르게 상수만 반환하는 식으로 구현을 많이했다.

이런 방식이 스텁구현이다. Dummy가 실제 동작하는것처럼 보이도록 한다.

## Spy

```java
public class MailingService {
    private int sendMailCount = 0;
    private Collection<Mail> mails = new ArrayList<>();

    public void sendMail(Mail mail) {
        sendMailCount++;
        mails.add(mail);
    }

    public long getSendMailCount() {
        return sendMailCount;
    }
}
```
Spy는 위와 같이 sendMailCount++를 통해 자신이 호출된 상황을 기록하는 객체이다.

## Mock

```java
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {
    @Mock
    private UserRepository userRepository;
    
    @Test
    void test() {
        // thenReturn을 통해 반환할 값을 지정
        when(userRepository.findById(anyLong())).thenReturn(new User(1, "Test User"));
        
        User actual = userService.findById(1);
        assertThat(actual.getId()).isEqualTo(1);
        assertThat(actual.getName()).isEqualTo("Test User");
    }
}
```

Mockito 프레임워크의 사용방법을 통해 알아보자면 findById를 통해 반환할 값을 직접 지정해줄 수 있다. 테스트시에 매우 편리하게 기대한 값을 반환할 수 있다.


