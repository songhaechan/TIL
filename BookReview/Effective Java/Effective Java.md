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

---

item # 5 : 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.
---
아이템 5의 첫문장 "클래스는 하나 이상의 자원에 의존한다". 이 문장이 의미하는 바는 객체지향과 디자인패턴에서 봤듯이 하나의 클래스는 다른 클래스의 인스턴스를 생성에 의존한다는 의미다.

```java
    public class SpellChecker{
        private static final Lexicon dictionary = new KoreanDictionary();
        private SpellChecker(){};
        // public static method ...
    }
```
SpellChecker클래스는 Lexicon클래스에 의존하고있다. 생성자도 private으로 막아서 싱글톤으로 생성했다.

별다른 문제가 없어보이지만 사전은 영어사전 한글사전 일본어사전 등등 다양하게 존재하고 클래스내부에서 final 필드로 객체를 담고있기때문에 변경에 유연하지않다.

사전을 정말 딱 하나 의존하고있다면 상관없겠지만 **다른 인스턴스를 다뤄야하는 경우**엔 의존객체주입을 사용하자.

```java
    public class SpellChecker{
        private static final Lexicon dictionary;

        public SpellChecker(Lexicon){
            this.dictionary = dictionary;
        };
        // public static method ...
    }
```

생성자를 통해서 외부에서 인스턴스를 받아와 사용할 수 있다. 받아온 인스턴스는 불변을 보장받고있기때문에 안전하기도하다.

정적 유틸리티 클래스와 싱글턴이 좋지않다는게 아니라 책에서도 클래스가 자원을 하나이상 사용할 때에는 의존객체를 이용하여 유연성을 얻을 수 있다라고 되어있으니 적절하게 판단을 해야할 것 같다.

**질문 p.29  이 패턴의 쓸만한 변형으로, 생성자에 자원 팩터리를 넘겨주는 방식이 있다.**

---

item # 6 : 불필요한 객체 생성을 피하라
---
불필요한 객체 ? 어떤 경우에 객체가 불필요해진다는 것일까?

```java
    String s = new String("java");
```

```java
    String s = "java";
```

자바의 정석에서도 강조했던 내용이다. java라는 단어를 새롭게 객체로 생성해서 쓰는 방식이있고, 문자리터럴을 상수화하여 저장한 방식이 있다.

굳이 같은 단어를 계속해서 새로운 객체로 만들 필요는 없다. 두 번째 방식은 재사용성이 매우 높다.

이와같이 불필요한 객체를 생성하게되면 비용이 발생하게되고(성능상에서의 비용) 이러한 비용을 줄이는게 이번 장의 핵심 내용이다.

```java
    static boolean isRomanNumeralSlow(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }
```
이 메서드는 내부에서 정규표현식용 Pattern인스턴스를 생성하고 한 번 사용되면 버려진다. Pattern은 인스턴스 생성비용이 높다고 소개하고있다.

이런 경우에 Pattern인스턴스를 내부에 불변으로 선언해놓고 재사용한다면 성능이 개선될 것이라는건 직관적으로도 와닿는다.
```java
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }
```

Pattern인스턴스를 내부에서 한 번 생성하고 isRomanNumberalFast가 호출될때마다 재사용될 것이다.

성능은 6.5배 정도 빨라진다!

또 다른 예로 오토박싱을 예로들고있다.

```java
     private static long sum() {
        Long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;
        return sum;
    }
```
이 예시를 보고 과연 정말 이렇게 코드를 작성하는 일이 있을까? 싶었다.

매우 극단적인 비효율성을 보여준다. Long 객체는 for문에 의해 231개가 생성된다.

하지만 한 가지 주의해야할 점이있다. 아이템 50에 방어적복사라는 개념이 나오는데 방어적복사는 객체의 재사용이 불러오는 피해를 제시하고 있다.

이번 아이템과 상당히 대조적인 아이템인데 핵심은 객체가 새로워야하느냐 재사용해야하느냐에 있는데 이 기준을 구분하는 일은 상당히 어려울것같다... 책에서도 매번 내가 생성하는 인스턴스의 생성비용을 정확하게 측정하는 일은 어렵다고 소개한다.

경험이 쌓인다면 생길거라 기대해본다.

---

item # 7 : 다 쓴 객체 참조를 해제하라
---

C에서는 동적할당을 통해서 메모리를 늘렸다 줄였다 혹은 해제(free)가 가능하다. 객체지향 수업시간에 배우는 C++도 소멸자가 있어서 해제(delete)가 가능하다.

자바는 GC가 사용하지않는 메모리는 해체해주기때문에 어떻게 보면 편하지만 편한 만큼 메모리관리에 소홀해질 수 있다.

확실히 C와 C++을 작성할땐 메모리 활성상태와 비활성상태를 자연스레 고려하게되지만, Java는 GC에 기대게된다.

```java
    public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
```

처음엔 도대체 메모리의 누수가 어디에서 일어나는지 확인하지 못했다.

스택이란 자료구조는 pop과 push를 size라는 변수를 통해 **논리저**으로 구현된다는 점이다.

pop과 push를 하더라도 논리적으로만 스택의 영역이 줄어들고 늘어나는거지 실제론 논리적인 스택영역 밖에는 객체가 존재하게된다.

문제점은 여기서 발생하는데 GC는 스택 활성영역 밖에있는 객체를 회수 할 수가 없다. GC는 활성영역밖에 있는 객체도 참조되고있다고 알기때문이다.

그래서 pop을 아래와 같이 구현해줘야한다.

```java
     public Object pop() {
       if (size == 0)
           throw new EmptyStackException();
       Object result = elements[--size];
       elements[size] = null; // 다 쓴 참조 해제
       return result;
   }
```
pop을 수행한 인덱스의 객체는 null로 해제해준다. 

null처리를 해주게되면 GC는 메모리회수대상으로 인식하게되므로 메모리의 누수가 발생하지않는다.

**질문 p.38 캐시 역시 메모리 누수를 일으키는 주범이다**

---

item # 8 : finalizer와 cleaner 사용을 피하라
---
finalizer는 자바 9에서 deprecated로 지정되었다. 그 대안으로 cleaner를 제시했지만 이 책에선 여전히 두 메서드 모두 예측불가능하고 성능상에 심각한 문제를 일으킨다고 강조한다.

자바는 GC에의해 메모리를 수거해간다. finalizer나 cleaner도 그저 GC에 해당 자원이 회수대상임을 알려줄 뿐 우선순위에 어떤 영향도 없다.

스레드를 배울 때에도 비슷한 내용이 있었다. 스레드는 OS 스케쥴러가 할당한 우선순위에따라서 실행되기때문에 스레드에 직접 우선순위를 지정해줘봐야 결국은 OS스케쥴러의 알고리즘이 제어한다.

같은 맥락으로 메모리 회수는 전적으로 JVM 알고리즘의 역할이고 finalizer와 cleaner는 회수대상을 알려주는 것 그 이상의 일을 할 수가 없다.

특히 finalizer는 예외가 발생해도 무시되고 처리할 작업이 남아있어도 종료된다.
그런데 미완성인 객체를 다른 스레드가 사용하려한다면 심각한 오류를 낳을 것이다.

그나마 cleaner는 자신의 스레드를 통제한다.

이런 문제점들을 안고있기때문에 그 대안으로 AutoCloseable를 구현ㄴ하고 close메서드를 호출해준다. close로 해당자원을 이제 더이상 사용하지않는 자원이라 명시해준다. finalizer와는 달리 예외를 발생시키기때문에 적절하게 예외처리를 통해서 작성해주면된다.

그럼에도 cleaner와 finalizer가 존재하는 이유에대해 알아보면, 개발자가 close를 작성하지않았다면 그에대한 안전망으로 늦게나마 회수를 해달라는 요청을 할 수 있다는 점이다.

정리 : finalizer는 사용하지말고(java 9이후부터) cleaner는 네이티브 자원 회수용으로만 사용 (이 경우에라도 자원의 중요성과 성능저하는 감수해야한다.)

---
item # 9 : try-finally보다는 try-with-resources를 사용하라
---

try-finally의 예제코드부터 살펴보자.

```java
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
```

위 코드에서는 InputStream 과 OutputStream 으로 자원을 두가지를 사용하고있다.

책에서는 기기에 물리적문제가 생겨 read메서드가 실패하고 당연히 close도 실패한다는 가정을 하고있다.

즉 try와 finally블록 모두에서 예외가 발생하는 경우 두번째 예외가 첫번째 예외를 삼키고 첫번째 예외에관한 어떤 정보도 남기지않는다.

try-with-resources를 이용해 개선한 코드를 보자.

try-with-resources는 try블럭 끝에 도달하면 try()내부에있는 자원을 자동으로 닫아준다.

```java
    static void copy(String src, String dst) throws IOException {
        try (InputStream   in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
```

위 코드에서 개발자 입장은 read메서드에서의 예외가 중요하지 close에서 예외는 부차적이다. 그래서 close메서드의 예외는 숨겨지고 read의 예외정보를 볼 수 있는데 close메서드의 예외도 스택 추적 내역에 숨겨져있다는 꼬리표를 달고 출력된다.

이렇게 try-with-resources는 코드가 보기 편할 뿐만아니라 예외의 정보도 훨씬 유용하다. 꼭 회수해야하는 자원을 다룰땐 try-with-resources를 사용하자.











