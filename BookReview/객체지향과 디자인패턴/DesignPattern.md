주요 디자인패턴
==
전략(Strategy)패턴
--

If-Else문이 길어지는 경험을 해본적이 있다. 모든 경우에 적용이 가능한 패턴은 아니지만 디자인패턴이라는것 자체가 allround-player일 수는 없다.

전략패턴도 앞서 살펴본 개방폐쇄원칙을 따른다. 

예시를 살펴보자.


```java
public class Calculator {
    public int calculate(boolean firstGuest, List<Item> items) {
        int sum = 0;
        for(Item item : items) {
            if(firstGuest) {
                sum += (item.getPrice() * 0.9);     // 첫 손님 10% 할인
            } else if(item.isFresh()) {
                sum += (item.getPrice() * 0.8);     // 덜 신선한 상품은 20% 할인
            } else {
                sum += item.getPrice();
            }
        }
        return sum;
    }
}
```

첫 손님에대한 할인과 그렇지 않은 손님에대해 if분기문을 사용해 로직을 구성했다.

이제껏 객체지향설계를 공부하며 숱하게 봐왔지만 새로운 요구사항이 발생한다면 위 if분기문을 수없이 많이 수정해야할 것이다.

변화가 발생하는 부분은 어떻게 처리해왔는지 생각해본다면 당연 인터페이스를 이용해 **분리**해야한다.
--

우선 분리된 결과부터 확인해보자.


```java
public class Calculator {
    private DiscountStrategy discountStrategy;
    public Calculator(DiscountStrategy discountStrategy) {
        this.discountStrategy = discountStrategy;
    }
    public int calculate(List<Item> items) {
        int sum = 0;
        for(Item item : items) {
            sum += discountStrategy.getDicountPrice(item);
        }
        return sum;
    }
}
```

개방폐쇄원칙을 지키기위해 의존역전원칙을 적용한 모습이다.
--

고수준모듈에서 추상타입으로 분리해 저수준모듈은 추상타입을 이용해 의존을 역전시킨것이다.

DiscountStrategy라는 인터페이스를 생성하고 그 하위에 콘크리트클래스를 생성한다. (FirstGuestDiscountStrategy)

Calculator 클래스는 그 어떤 수정없이도 기능은 비슷하지만 다른 로직을 수정없이 계산해낼 수 있다.

마지막 손님은 20퍼센트를 더 할인해준다는 새로운 정책이 생긴다면 DiscountStrategy를 구현하는 LastGuestDiscountStrategy 클래스를 생성하고 생성자에 넘겨주면 그만이다. (확장에 열려있다.)

Calculator를 사용하는 입장을 알아보자.
--

```java
public void 가격정책에 따른 가격계산(){
    Calculator cal = new Calculator(전략);
    int price = cal.calculate(items);
}
```
정말 정말 아름다운 코드가 돼버렸다...

확장에는 무긍무진한 가능성이 열려있고, Calculator 클래스는 그 어떤 확장에도 수정이 가해지지않는다.

정리하자면 전략패턴이란 **동일한 기능이지만 내부 로직에따른 미세한 차이점을 인터페이스를 이용해 분리해낸다**
--

여기서 DiscountStrategy를 **전략** 이라 부르고 직접 가격계산로직을 담당하는 클래스를 **맥락** 이라 부른다.

---

템플릿 메서드
--

전략패턴은 동일한 기능이지만 내부 로직이 다른 경우지만 템플릿 메서드패턴은 동일한 절차를 가진 클래스들을 객체지향적으로 설계하는 방법이다.

동일한 절차를 가진 두 클래스
--
```java
public class DbAthenticator {
    private UserDao userDao;
    public DbAthenticator(UserDao userDao) {
        this.userDao = userDao;
    }
    public Auth authenticate(String id, String pw) throws Exception {
        User user = userDao.selectById(id);
        boolean auth = user.equalPassword(pw);
        if(!auth) {
            throw new Exception("has not auth");
        }
        return new Auth(id, pw);
    }
}
public class LdapAuthenticator {
    private LdapClient ldapClient;
    public LdapAuthenticator(LdapClient ldapClient) {
        this.ldapClient = ldapClient;
    }
    public Auth authenticate(String id, String pw) throws Exception {
        boolean auth = ldapClient.authenticate(id, pw);
        if(!auth) {
            throw new Exception("has not auth");
        }
        return new Auth(id, pw);
    }
}
```

위 두 클래스는 사용자의 인증을 두 가지 방식으로 제공하고있다.

하지만 절차는 같다. 인증이 실패하면 exception을 던지는 것과 인증이 성공하면 인증된 결과 인스턴스를 반환한다.

그렇다고 전략패턴을 이용해 각자에게 구현을 맡긴다면 위 모습과 그리 다르지않을 것이다.( 어차피 재정의는 자신에게 맞게 바꿔야하기때문이다. )

절차도 유지하고 코드의 중복도 제거하고싶을 때 템플릿 메서드 패턴을 이용하면 된다.

이 때 해당 절차를 하나의 틀이라 생각하고 추상클래스로 구현한 뒤에 하위 클래스에게 각각 알맞은 재정의를 강제하면된다.

틀이라는 단어가 영어로 Template이고 이러한 의미로 이름지어진 패턴이다.

템플릿 메서드 패턴 적용
--
```java
public abstract class Authenticator {
    public Auth authenticate(String id, String pw) throws Exception {
        if(!doAuthenticate(id, pw)) {
            throw new Exception("has not authentication");
        }
        return createAuth(id);
    }
    protected abstract boolean doAuthenticate(String id, String pw);
    protected abstract Auth createAuth(String id);
}
```
인증실패 시 예외를 던지고 마지막에 인증이 확인되면 인증 인스턴스를 반환하는 부분을 그대로 구현했다. (추상메서드가 아님)

하지만 구현체에따라 달라지는 부분, 즉 doAuthenticate과 createAuth는 각자에게 맡긴 모습이다.

protected인 이유는 외부에서는 해당 메서드를 호출할 일을 만들지 않기위함이자 재정의를 가능하게 하기위함이다.

새로운 요구사항이 발생하면 똑같은 코드를 또 작성하지않아도되고 기존의 코드중복도 사라졌다.

또 중요한 점은 상위 클래스가 흐름을 제어한다는 점이다.
--

절차라는 큰 흐름은 상위클래스가 갖고 하위클래스는 구체 기능구현만을 제공한다. 흐름의 변경은 오직 상위클래스만이 하게된다.

또한 위 예시는 템플릿 메서드에서 호출하는 메서드는 추상메서드로 정의했지만 abstract 메서드가 아닌 비어있는 기본구현메서드를 제공해 **하위클래스가 기능의 확장**을 할 수 있도록 설계할 수 있다.

이런 메서드를 훅 메서드라고 부른다.

템플릿 메서드의 변형
--

템플릿 메서드는 상속을 이용해 기능을 제공하지만, 상속은 불필요한 재사용을 증가시킬 수 있고, 이펙티브자바에서 소개한 대로 equals의 재정의를 불가능하게만든다. 결정적으로 런타임에 객체를 변경할 수 없다.

하지만 아래 예시를 보면 메서드를 파라미터로 받아 큰 흐름 속에 원하는 메서드를 삽입할 수 있다.

```java
public <T> T execute(ConnectionCallback<T> action) throws DataAccessException {
      Assert.notNull(action, "Callback object must not be null");
      Connection con = DataSourceUtils.getConnection(obtainDataSource());

      try {
            Connection conToUse = createConnectionProxy(con);
            return action.doInConnection(conToUse);

        } catch (SQLException ex) {
            // 생략

        } finally {
            // 생략

        }

    }
```

(사실 해당 패턴은 전략패턴에 더 가깝다. )

하지만 위 방식은 훅 메서드를 이용해 확장 기능을 제공하려면 복잡해지는 단점이 있다.

---

상태패턴
--

상태패턴은 조건에따라 상태가 달라지는 분기문을 객체지향적으로 설계하는 디자인패턴이다.

기존의 분기문 코드
--
```java
public class VendingMachine {
    // ...
    public void insertCoin(int coin) {
        switch (state) {
            case NOCOIN:
                // ...
            case SELECTABLE:
                // ...
            case SOLDOUT:
                returnCoin();
        }
    }
    public void select(int id) {
        switch (state) {
            case NOCOIN:
                // ...
            case SELECTABLE:
                // ...
            case SOLDOUT:
                // Nothing
        }
    }
    // ....
    private void returnCoin() { }
    private enum State {
        NOCOIN, SELECTABLE, SOLDOUT;
    }
}
```

위 예시는 자판기를 구현했는데 동전을 넣었을 때 상태가 동전이 없는 경우 제품이 선택 가능한 경우로 나눈다.
제품을 선택하는 경우에도 마찬가지로 분기분이 형성된다.

문제는 새로운 조건이 들어올 때마다 분기문을 다 찾아서 수정해줘야한다는 점이다.

바로 이 부분이 변화가 일어나는 부분이고 새로운 요구사항에 계속해서 변화가 생긴다.

이런 경우에 우린 개방폐쇄원칙을 적용할 수 있다.

VendingMachine은 고수준모듈로써 추상화된 모듈을 생성하고(인터페이스) 저수준 모듈(상태변화 객체)은 이 추상화모듈에 의존한다.(의존 역전 원칙)

상태패턴 적용
--
```java
public class VendingMachine {
    private State state;
    // ...
    public VendingMachine() {
        state = new NoCoinState();
        // ...
    }
    public void insertCoin(int coin) {
        state.increaseCoin(coin, this);     // 상태 객체에 위임
    }
    public String select(int id) {
        return state.select(id, this);     // 상태 객체에 위임
    }
    void changeState(State newState) {
        this.state = newState;
    }
    // ...
}
```

이렇게 인터페이스타입으로 기능을 하위객체에 위임하고 VendingMachine은 새로운 기능추가에 적은 영향을 받게된다. 아예 영향을 받지않을 수는 없겠지만 기존 코드에서 분기문을 처리하는 일보다는 간단하다.

NoCoinState 객체
--
```java
public class NoCoinState implements State {
    @Override
    public void increaseCoin(int coin, VendingMachine vendingMachine) {
        vendingMachine.increaseCoin(coin);
        vendingMachine.changeState(new SelectableState());
    }
    @Override
    public String select(int productId, VendingMachine vendingMachine) {
        return "코인이 존재하지 않습니다.";
    }
}
```

동전이 없는 상태일 때 하위객체에서 직접 동전을 증가시키고 **상태를 변경한다**

상태 변경의 주체에대해서도 알아봐야한다. 위 예시에선 하위클래스가 직접 상태를 변경시키는데 VendingMachine에서도 상태를 변경시킬 수 있다.

고수준모듈에서 직접 상태를 변경하도록하면 자연스럽게 수정이 많아질 수 있다. 고수준에서 직접 제어를 한다는 개념자체가 변화가 발생하면 고수준의 수정이 불가피하기때문이다.

그렇기때문에 상태의 변경이 거의 없는 경우 상태의 개수가 변경되지않을 경우에 고수준에서 제어하도록해야한다.

**개인적인 생각인데 고수준에서 상태변경은 개방폐쇄원칙을 명확하게 지키지못함으로 좋은 방식은 아닌듯 하다.**

하지만 저수준에게만 상태제어를 맡기는 것도 단점은 존재한다. 상태가 늘어나게되면 그만큼 클래스도 늘어날 것이고 하위 클래스 서로가 서로를 의존하기때문이다.

---

데코레이터 패턴
--

SOLID원칙을 알아보기전에 **상속을 조심히 사용해야하는 이유**에대해 알아보았다.

기존에 파일을 출력해주는 클래스가 있다고 해보자. 그런데 새로운 요구사항으로 압축후 파일을 출력하거나 암호화후에 파일을 출력해야한다면 기존 클래스를 상속받아 기능을 확장할 수 있을 것이다.

그런데 만약 암호화 후에 압축, 압축 후에 암호화 등 각 기능을 합친 새로운 기능의 요구는 클래스 계층을 복잡하게 만들뿐더러 불필요한 클래스가 증가한다.

데코레이터 패턴은 각각의 기능을 조합하게 끔 만들어주는 추상클래스로 정의된다.

불필요한 클래스를 생상하지않고 기존의 기능을 조합해서 사용한다고 생각하면 되겠다.

    FileOut<<Interface>> <------- Decorator<<Abstract>>
    ^
    |                               ^          ^       ^
    |                               |          |       |
    |                               |          |       |
    FileOutImpl           BufferedOut   EncryptionOut  ZipOut

Decorator 와 하위클래스
--
```java
public abstract class Decorator implements FileOut{
    private FileOut delegate; // 위임 대상
    public Decorator(FileOut delegate){
        this.delegate = delegate;
    }

    protected void doDelegate(Byte[] data){
        delegate.write(data);
    }
}

//하위 클래스
public class EncryptionOut extends Decorator{
    public EncryptionOut(FileOut delegate){
        super(delegate);
    }
    public void wirte(Byte[] data){
        byte encryptedData = encrypt(data);
        super.doDelegate(encryptedData);
    }
}
```

이제 호출하는 쪽 코드를 살펴보고 흐름을 알아보자.
--

```java
FileOut delegate = new FileOutlmpl();
FileOut fileOut = new EncryptionOut(delegate);
fileOut.write(data);
```

Client의 목적은 암호화한 뒤에 파일에 쓰려는 것이다.

**우선 마지막 위임대상은 파일에 쓰는 것임을 명심하자.**

암호화객체에 FileOutlmpl객체를 넘겨준다.

암호화객체 내부에서 처음으로 write 메서드를 호출하면 넘겨받은 데이터를 암호화한 뒤 super.doElegate()메서드를 통해 암호화한 데이터를 넘겨준다.

최종적으로 delegate객체가 write메서드를 호출함으로써 작업은 완료된다.

데코레이트 패턴의 묘미는 여러가지 작업을 순서대로 처리할 때 빛을 발한다.

```java
FileOut fileOut = new BufferedOut(new EncryptionOut(new ZipOut(delegate)))
```

위 코드를 그대로 따라가보자.

1. BufferedOut 의 write 메서드 호출 후 데이터를 버퍼로 처리

2. 다음 위임상대에게 데이터를 넘겨주고 암호화객체 write호출

3. 암호화 후 압축 객체의 write 호출

4. 최종적으로 파일객체에 wirte 호출

이렇게 꼬리에 꼬리를 물며 각각의 기능을 더하고 더해 최종적은 결과물을 만든다.

주의할점
--
지금 데코레이터는 write한가지 메서드만 정의하고있지만 이 메서드가 증가하면 구현이 매우 복잡해질 가능성이 있다.

또한 구조가 복잡해지면서 write메서드를 단순히 호출한다면 그 과정을 모두 따라가며 이해해야 전체적인 이해가 가능하다.
(왜냐하면 클라이언트 코드에서 fileOut.write(data) 로만 호출하기때문)

---
프록시 패턴
--

우선 Proxy 란 대리, 대리인 이라는 뜻을 가졌다.

한글로 굳이 해석하자면 대리 패턴... 정도가 되겠다.

책에 나온 예시를 간단히 소개하겠다.

    제품목록을 보여주는 GUI 프로그램

    문제 의식 : 한 화면을 벗어나는 이미지까지 로딩하기엔 메모리낭비가 심하다.

    스크롤 할 때 이미지를 즉각적으로 로딩해준다면 메모리 낭비가 해결될 것이다.

    해결책 : 프록시 패턴을 이용하자!


    ListUi       ------>    <<Interface>> Image
                             ^              ^
                             |              |
                             |              |
                     ProxyImage  ----->   RealImage

실제 이미지를 로딩해서 가지고있는 객체는 RealImage 이다.

하지만 여기서 중요한 것은 당연 ProxyImage이다.

ProxyImage
--
```java
public class ProxyImage implements Image{
    private String path;
    private RealImage image;

    public ProxyImage(String path){
        this.path = path;
    }
    public void draw(){
        if (image == null){
            image = new RealImage(path);
        }
        image.draw();
    }
}
```

ListUI
---
```java
public class ListUI{
    private List<Image> images;
    public ListUI(List<Image> images){
        this.images = images;
    }

    public void onScroll(int start, int end){
        // image.draw()
    } 
}
```

중요한 것은 만일 List의 Image타입이 ProxyImage로 구성되어있다면 ListUI가 draw를 호출하기 전까지는 RealImage객체를 생성하지않는 다는 점이다. 

왜냐하면 RealImage의 객체생성 권한은 전적으로 ProxyImage에게 있고 제공 또한 ProxyImage가 직접한다.

여기서 또 다시 추상화가 엿보인다. List<Image>에서 Image는 위에 정의한 인터페이스다. 하지만 List에서 Image를 하나하나 꺼내 draw()메서드를 호출하더라도 Image의 실제객체가 ProxyImage인지 RealImage가 직접 들어있는지 상관하지 않아도된다. 

만약 이미지 로딩정책이 첫 4개 이미지는 바로 표출하고 그 외에는 사용자가 스크롤을 내려야만 표출하는 정책이라면 List에 RealImage객체와 ProxyImage객체를 섞어서 제공하면 된다.

만약 사용자가 스크롤을 내리지않는다면 ProxyImage의 draw는 호출되지않을 것이고 RealImage객체 또한 생성되지 않기때문에 메모리 낭비가 발생하지않는다.(ProxyImage 객체를 생성하더라도 RealImage객체는 생성되지않는다.)
--

























