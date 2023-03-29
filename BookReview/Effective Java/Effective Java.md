Effective Java
==
Effective Java를 읽고 정리하려한다.

1장
==

Subclassing? Subtyping?
--
>이펙티브자바에서 상속을 서브클래싱(subclassing)과 동일한 의미로 기술했다는 내용이나온다. 상속은 들어봤지만 서브클래싱이란 무엇일까?

간단히 말해 Subclassing은 코드의 재사용만을 목적으로하는 상속을 말한다. 

펭귄과 새를 예시로 들어보면자

> 펭귄 : [날개] , [걷기] , [부리] 등등

> 새 : [날개] , [걷기] , [날 수 있음] 등등

펭귄은 새와 공통점이 많지만 펭귄은 날 수가 없다. 하지만 나머지 특성들이 같아서 코드의 재사용을 위해 펭귄이 새를 상속받는 것을 서브클래싱이라고 한다.

개발자는 코드의 재사용만을 위한 상속은 지양하고, 타입계층의 구분을 위한 명확한 상속(Subtyping)을 지향해야한다.

메서드 시그니처
--
메서드의 이름과 입력 매개변수의 타입들로 이뤄진다.

클라이언트와 사용자
--
API를 사용하는 작성자를 API의 사용자라 하고, API를 사용하는 클래스를 그 API의 클라이언트라 한다.

2장
==
item #1 : 생성자 대신 정적 팩터리 메서드를 고려하라
--
public 생성자는 반활될 객체의 특성을 쉽게 파악하기 어렵다. 게다가 매개변수가 늘어나면 매개변수의 특성 또한 알아보기 쉽지않다.

```java
public static Boolean valueOf(boolean b){
    return b ? Boolean.TRUE : Boolean.FALSE
}
```

위는 책에서 제시한 대표적 정적 팩터리메서드다.

생성자를 이용하지않고 매개변수에 boolean을 넘겨주고 객체를 반환받는데, 메서드시그니처로 어떤 객체를 반환받을지 직관적으로 파악이 가능하다.

정적팩터리 메서드의 장단점을 알아보자.


**장점**

1. 이름을 가질 수 있다.

    대부분의 생성자는 객체의 특성을 보여줄만하다고 하는건 클래스의 이름뿐이다.

    하지만 정적 팩터리 메서드는 이름을 가질 수 있다.

    ```java
    BigInteger.probablePrime
    ```
    위의 메서드는 정적메서드인데 값이 소수인 BigInteger를 반환한다.
    이름으로 반환될 객체의 특성을 쉽게 파악할 수 있다.

2. 호출마다 인스턴스생성을 하지않아도된다.

    반환하는 객체에서 미리 인스턴스를 생성해놓고 정적 팩터리 메서드를 구현하면 같은 인스턴스를 계속해서 반환해준다. 즉 클래스를 싱글톤으로 만들 수 있다.

3. 반환 타입의 하위 타입 객체를 반환할 수 있다.

    슈퍼클래스의 서브클래스를 반환 할 수 있게된다. 사용자는 슈퍼클래스로 반환 받은 것 같지만 서브 클래스를 사용 할 수 있게된다.

    이는 인터페이스 기반 프레임워크의 핵심 기술 이다.

    반환타입은 인터페이스 타입으로 하고 실제 반환하는 객체는 구현체를 반환하도록 함으로써 구현내용은 숨길 수가 있다.

    자주 사용하는 Java.util.Collections에는 많은 메서드가 선언돼있지만, 우리는 구현체의 내용을 볼 수 없다.(non-public 이기때문)

        java 9 private method에 대해

        java 8 부터 인터페이스에 default메서드 사용이 가능해지고 public static method 사용이 가능해졌다. default는 해당 인터페이스에 메서드를 구현이 가능하게 해주고, public static method는 객체 생성없이 바로 호출이 가능해졌다.

        java 9 이후 private method 가 추가되었는데 왜 필요한지 알아보았다.
        아래 예시코드를 보자

        
        class Java9{
            public static todoSomething(){
                // 등교한다
                // 수업을 듣는다.
                // 공부를 한다
                // 잔다.
            }
            public static todoSomethingTomorrow(){
                // 등교한다
                // 책을 읽는다.
                // 술마신다.
                // 잔다.
            }
        }
        
        위 메서드에서 등교한다와 잔다는 행위는 중복돼있고 따로 분리한다.
        이 때 분리할 메서드가 과연 밖으로 노출될 필요가 있을까?
        todoSomething()과 todoSomethingTomorrow()에서만 사용하므로 노출될 필요가 전혀 없다.
        그래서 private static으로 외부로부터 접근을 제한해준다.

4. 입력 매개변수에 따라 매번 다른 크래스의 객체를 반환할 수 있다.

    말이 어렵다... 예시를 가지고 이해해보자.

    EnumSet 클래스는 정적 팩터리메서드만을 제공해주는데, 우리가 원소의 개수를 넘겨주면 해당 개수에 알맞은 인스턴스를 반환해준다.

    실제 내부에서는 원소 개수가 64개일때와 그 이상일 때 반환하는 인스턴스가 다르다.

    이처럼 메서드를 호출하는 사용자는 매개변수만 넘겨줄 뿐 반환되는 인스턴스가 어떤 인스턴스인지 알 수도 없고 알 필요도 없다.

    단순하게 내가 반환받을 클래스의 하위타입들이기만 한다면 아무런 문제가 없는것이다.

5. 정적 팩터리 메서드를 작성하는 시점에는 반활할 객체의 클래스가 존재하지 않아도 된다.

    역시 어렵다. 무슨 말인지 이해하는데 한참이 걸렸다.

    책에서 JDBC를 예시로드는데 해당 내용은 프레임워크를 어느정도는 알고 있어야 이해가 쉬울 듯 하다.

    JDBC에서 getConnection이라는 메서드가 있는데 해당 메서드는 인스턴스를 반환하기는 하지만, **중간에 어떤 DB가 연결되느냐에 따라 반환하는 인스턴스가 달라진다**. 처음엔 NoSQL이 연결되어 connection을 얻어왔다고 가정하자.

    이후에 다른 DB에 연결하고 싶어 DBdriver를 MySQL로 교체를 해주면 getConnection은 MySQL에대한 인스턴스를 반환한다.

    핵심은 메서드를 먼저 작성해 놓고 반환될 인스턴스는 나중에 결정할 수 있다는 점이다.


**단점**

1. 상속을 위해서는 생성자가 필요하다.
    

2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
    
    맞는 말이다. 나도 어떤 정적 메서드가 어떤 인스턴스를 반환하는지 그때그때 찾아봐야 알지 자주쓰는 메서드가 아니라면 쉽게 파악하기 어렵다.
    생성자는 API설명에 확실하기때문에 인지하기는 쉽다.

    그래서 정적 팩터리 메서드의 명명방식이 존재한다. (Official은 아님)


**정리**

1,2번은 단순하게 이름을 가질 수 있고, 싱글턴을 보장할 수 있다는 내용이다.
3,4,5번은 좀 더 유연한 프로그래밍을 할 수 있도록 도와준다. 구현내용을 외부에 공개하지않고 인터페이스만으로 개발이 가능하도록 하고, 메서드의 작성시점이 인스턴스 반환시점에 국한되지않게 만들 수 있다.

어렵다. 매우 어렵다. 공부를 열심히 하자.

---

item #2 : 생성자에 매개변수가 많다면 빌더패턴을 고려하라
---

생성자에 매개변수가 많으면 외부에서 생성자를 호출하고 객체를 생성할 때 내가 넘겨주는 매개변수가 어떤 멤버에 값을 넘겨주는지 알기 어렵다.

매개변수가 2-3개라면 문제가 없겠지만, 많아질 경우 코드를 읽기 불편하고, 작성할 때에도 파악이 어려워진다.

이 때 빌더패턴이란 것을 고려하자.

우선 빌더는 사용할 클래스의 안에 public static 클래스로 내부클래스를 생성해준다.

```java
public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }
```

필수 매개변수는 빌더의 생성자로 넣어주도록하고 builder클래스 내부에 setter메서드를 생성한다.

```java
        public Builder calories(int val)
        { calories = val;      return this; }
        public Builder fat(int val)
        { fat = val;           return this; }
        public Builder sodium(int val)
        { sodium = val;        return this; }
        public Builder carbohydrate(int val)
        { carbohydrate = val;  return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
```

위 메서드들의 반환타입을 보면 **Builder를 반환하면서 연쇄적으로 호출**이 가능하다.

생성자 호출 후 setter메서드를 연쇄적으로(선택적으로) 호출하고 마지막에 반환타입이 NutritionFacts인 build()메서드를 호출해 최종적으로 원하는 인스턴스를 반환한다.

```java
return new NutritionFacts(this);
```

여기서 this는 Builder 객체를 의미하고 전달받은 Builer객체를 NutritionFacts의 private생성자에 전달한다.

```java
private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
```

해당 생성자는 외부에 노출이 불필요하므로 private으로 생성하고 위에서 설명했듯이 builder 인스턴스를 받아와 NutritionFacts의 멤버를 설정한다.

```java
        NutritionFacts cocaCola = new NutritionFacts.Builder(240,8).calories(100).sodium(35).carbohydrate(27).build();
```

이제 어떤 멤버에 어떤 값을 지정할 지가 명확해졌다.

>뒷 내용중 인터페이스를 활용해 추상화로서의 장점이 있지만, 이해하기가 너무 어렵다... 백기선님 강의를 참고해서 후에 다시 정리하자.

---
item #3 : private생성자나 열거 타입으로 싱글턴임을 보증하라
---

싱글턴이란 호출횟수에 상관없이 단 하나의 인스턴스가 반환됨을 말한다.

해당 내용을 정확히는 이해하지못한것 같지만, 우선은 정리해보자.

우선 싱글턴을 만드는 방법은 두가지인데, 두 방식 모두 생성자는 private으로 막아두고 public static 멤버를 하나 만들어두고 해당 멤버에 private 생성자로 단 한번 인스턴스를 반환한다.

```java
    public class A{
        public static final A INSTANCE = new A();
        private A(){};
    }
```
A.INSTANCE 외에는 인스턴스를 생성할 수 없다.
> 리플렉션 API인 AccessibleObject.setAccessible로 private을 사용할 여지가 남아있지만, 생성자에 두 번째 객체가 생성될 때 예외를 던진다. ( AccessibleObject.setAccessible는 후에 알아보자.)

이 방식의 장점은 첫 째로 간결하다.

그리고 final로 멤버가 선언되어있기때문에 INSTANCE에 절대 새로운 객체를 할당할 수가 없다. 명백하게 싱글턴임이 보증된다.

싱글턴을 만드는 두 번째 방식은 item1에서 언급한 정적 팩터리 메서드다.

> 정적 팩터리로 싱글턴을 보장하는 장점 3가지가 나오지만 이해하기가 어렵다. 이 내용도 뒤로 미뤄두자...

싱글턴을 만드는 세 번째 방식은 원소가 하나인 열거타입을 선언하는 것이다. 

>해당 내용도 이해가 하나도 되지않는다. 리플렉션, 직렬화가 무슨 뜻인지 모르겠다. 학습하자.

---
item #4 : 인스턴스화를 막으려거든 private생성자를 사용해라
---

내가 생성한 클래스의 인스턴스화를 막고싶은 경우가 생긴다고하는데 아직은 와닿지는 않는다.

Math클래스나 Arrays클래스를 예시로 들었다. 항상 랜덤숫자를 생성할 때 자주 사용하곤하지만, Math클래스를 상속받거나 인스턴스화해서 사용하지는 않는다.

이럴때 생성자를 private으로 만들어주면 클래스의 인스턴스화를 원천적으로 막을 수 있다.

또한 생성자가 private하기때문에 상속또한 불가능하게 만들어 줄 수 있다. (상속을 위해선 기본생성자가 필요하기때문)

생성자를 아예 없애버리면 컴파일러가 기본 생성자를 만들어주기때문에 인스턴스를 막기위해선 꼭 private으로 선언해줘야한다.

---

item + : 직렬화 역직렬화 리플렉션
--

    Effective Java를 읽으며 직렬화, 역직렬화, 리플렉션이라는 개념이 나왔고 이를 간단하게 멘토님께서 설명해주셨다.

    간단하게 정리해보려한다.

    >객체를 직렬화한다는 건 객체 상태를 바이트 스트림으로 바꿔서 바이트 스트림을 객체의 복사본으로 되돌릴 수 있음을 의미한다. (중략) 역직렬화는 직렬화된 객체 형식을 객체의 사본으로 다시 바꾸는 프로세스다.

    위 내용은 코틀린 공식문서중 한 문장이다.

    우리가 JSON방식으로 객체를 클라이언트에게 내릴때 JSON을 사용하곤 한다.

    이러한 방식을 **역직렬화**라 한다.

    다시 바이트 스트림형식을 다시 객체형식으로 되돌리는 것을 **직렬화**라한다. 클라이언트에게 데이터를 전송받아 서버에서 객체를 생성하는 과정을 말한다.

    **리플렉션이란** 구체적인 클래스의 타입을 알지 못하더라도 클래스의 메서드, 필드에 접근 할 수 있도록 도와주는 것을 말한다.

    프레임워크에서 자주 쓰이는 개념으로, 프레임워크는 우리가 생성하는 클래스를 작성시점에 알지 못하므로 런타임시에 동적으로 클래스의 정보를 추출할 수 있게해준다.











