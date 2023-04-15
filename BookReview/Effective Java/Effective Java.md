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

---
item # 10 : equals는 일반 규약을 지켜 재정의하라
---
Object클래스의 equals메서드를 재정의할 때는 많은 주의가 필요하다. 재정의속에서 많은 함정이 도사리고있기때문에 문제를 피하기위한 최고의 방법은 "재정의하지 않는 것"이다.

1. 각 인스턴스가 본질적으로 고유하다. 

    이 책이 어려운 이유는 말이 어렵다... 인스턴스가 본질적으로 고유하려면 본질적으로 값을 가지는 클래스가 아니여야한다. 논리적 동치성(Equality)를 생각조차하지않아도 되는 Thread가 그 예시다.

2. 논리적 동치성을 검사할 일이 없다.

    설계시에 논리적 동치를 아예 필요로하지 않는다면 재정의할 필요가 없다. 이 경우 Object의 기본 equals로 해결한다.

3. 상위 클래스에서 재정의한 equals가 하위클래스에도 딱 들어맞는다.

    뒤에서 자세히 다루겠지만 하위클래스에서 상속받은 필드를 제외하고 새로운 필드를 새롭게 생성하면 논리적 동치성 또한 다시 정의해야한다. 결론은 하위 클래스에서 재정의 한 equals는 equals의 동치관계라는 규칙을 절대로 지킬 수가 없음으로 이 상황은 하위클래스가 새로운 필드를 전혀 생성하지않았다는 상황으로 이해하면 좋다.

4. 클래스가 private이거나 protected이고 equals가 호출할일이 없다.

    이런 경우엔 equals 호출 시 예외를 던진다.

그렇다면 언제 equalsf를 재정의 해야할까?

예를 들어, Student클래스에 필드가 이름과 나이 학번이 있다. 그런데 인스턴스 중에 이름 나이 학번이 모두 같은 객체가 있다면 상식적으로 같은 객체로 판단하고 싶을 것이다.

이럴 때 equals를 재정의한다. 책에서 매우 강조하지만 **일반 규약**을 따라 재정의 해야한다.

모든 참조값 x,y에대해 아래를 만족해야한다.
1. 반사성 x.equals(x) = true 
2. 대칭성 x.equals(y) = true 면, y.equals(x) 도 true
3. 추이성 x.equals(y) = true 이고 y.equals(z) 면, x.equals(z) = true 이다.
4. 일관성 x.equals(y) 의 반복호출에도 true여야한다.
5. null-아님 x.equals(null)은 false다.

반사성과 일관성 null-아님은 지키는 것보다 못지키는 것이 더 어렵다.

대칭성과 추이성에대해 알아보자. 

대칭성이다.
--
```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배!
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
}
```
CaseInsensitiveString타입의 "Polish"와 String s = "polish"가 있다.

cis.equals(s)는 두 번째 if문으로 들어가 대소문자를 무시하고 true가 리턴된다.

그렇다면 s.equals(cis)는 어떨까? 애초에 String클래스가 CaseInsensitiveString으로 변환이 될까?

절대 변환 될 수 없음으로 항상 false가 동작한다.

두 번째 규약 대칭성을 명백히 위반한다.

```java
@Override public boolean equals(Object o) {
       return o instanceof CaseInsensitiveString &&
               ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
   }
```
애초에 String은 이 equals 메서드에서 수행될 수 없도록 만든 코드다.

다음은 추이성이다.
--

equals를 재정의하지 않아도 되는 상황 4번째에서 언급했듯이 상위클래스의 equals가 하위클래스에게 딱 들어맞으면된다.

하지만 앞으로 설명하겠지만, 하위클래스에서 새로운필드를 생성하는 순간 equals 재정의가 불가능 해진다.

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
}
```
아주 자연스럽게 x y를 필드로하고 해당 클래스에 equals를 재정의했다.

문제는 Point를 상속받는 ColorPoint의 생성이다.

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
}
```
이제 새로운필드를 갖는 하위클래스가 생겼다. 기존 equals는 x,y에 대해서만 논리적동치를 가려낸다. 이 클래스의 논리적 동치는 어떻게 해야할까?

```java
@Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```
이 코드도 얼핏보면 규약을 다 지킨 것만 같다. x,y의 동치는 상위클래스에 맡기고 color만 하위클래스에서 따지겠다는 말이다.

예시로 시험해보자!

Point p = new Point(1,2);
ColorPoint cp = new ColorPoint(1,2, Color.RED);

p.equals(cp) 는 true가 나올 것이다. 하지만 cp.equals(p)는 o instanceof ColorPoint가 false이므로 false이다.

하지만 대안이 있긴하다 (틀렸지만) 일반 Point가 들어오면 색은 무시하고 x y만 비교하는 방법이다.

```java
@Override public boolean equals(Object o) {
       if (!(o instanceof Point))
           return false;

       // o가 일반 Point면 색상을 무시하고 비교한다.
       if (!(o instanceof ColorPoint))
           return o.equals(this);

       // o가 ColorPoint면 색상까지 비교한다.
       return super.equals(o) && ((ColorPoint) o).color == color;
```
기존에는 o instanceof ColorPoint 가 false를 반환하고 return false를 했지만 이번엔 색을 무시하고 상위클래스의 equals로 비교하기 시작한다.

이렇게하면 대칭성은 만족하지만 추이성을 만족하지 못한다!

```java
ColorPoint p1 = new ColorPoint(1,2,Color.RED);
Point p2 = new Point(1,2);
ColorPoint p3 = new ColorPoint(1,2,Color.BLUE);
```
p1.equals(p2) 와 p2.equals(p3) 는 참이지만 p1.equals(p3)는 거짓이므로 추이성을 만족하지 못한다.

**즉, 하위클래스에서 값을 추가하면서 equals도 재정의하는 방법은 존재하지않는다.**

왜 추이성이 성립해야하는지 수학적으로 설명하자면 p->q , q->r 이면 p->r이 성립한다. (이에대한 증명은 TruthTable을 사용하면 증명할 수 있다.)

```java
   // 잘못된 코드 - 리스코프 치환 원칙 위배! (59쪽)
   @Override public boolean equals(Object o) {
       if (o == null || o.getClass() != getClass())
           return false;
       Point p = (Point) o;
       return p.x == x && p.y == y;
   }
```
위 처럼 equals를 재정의하면 값도 추가하면서 하위클래스를 확장할 수 있을까?

가능하지만 리스코프 치환원칙을 위배한다. 

>리스코프치환원칙이란 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다. 따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야한다.

ColorPoint도 여전히 정의상 Point이므로 어디서든 Point로써 활용가능해야하지만, getClass()로는 같은 구체클래스만이 if문을 통과하기때문이다.

하지만 방법이 없는 것은 아니다. **객체지향과 디자인패턴**에서도 언급했듯이 상속을 지양하고 조립을 이용하라는 개념이 있었다.

ColorPoint가 Point를 상속하지않고 필드에 주입받아 사용하는 조립을 이용하면 간단하게 해결이 가능하다. (상속을 이용해 equals를 규약을 지키며 재정의할 수는 없다)

결론을 짓자면 equals는 최대한 재정의하지 말자. 왜냐하면 Object나 제공해주는 클래스에는 이미 적절한 재정의가 이루어져있고, 꼭 필요하다면 대칭성 추이성 일관성 을 지켜가며 재정의하자.

---
item # 11 : equals를 재정의하려거든 hashCode도 재정의하라.
---

왜 equals를 재정의할 때 hashCode도 재정의 해야할까 그 이유부터 알아보자.

```java
public class Car {
    private final String name;

    public Car(String name) {
        this.name = name;
    }
    //equals만 재정의
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Car car = (Car) o;
        return Objects.equals(name, car.name);
    }
}
```

> item 10에서 언급했듯이 equals를 getClass()방식으로 비교하면 안된다. (리스코프치환원칙을 위배하기때문에) 이 예시는 Car를 상속하는 클래스가 없음으로 getClass()로 재정의했다.

name이 동일한 Car인스턴스를 List타입에 넣게되면 인스턴스는 2개가 생성된다. List는 중복을 검사하지않기때문이다.

하지만 아래내용을 보자.

```java
public static void main(String[] args) {
    Set<Car> cars = new HashSet<>();
    cars.add(new Car("foo"));
    cars.add(new Car("foo"));

    System.out.println(cars.size());
}
```
Set은 중복을 검사해 같은 인스턴스가 Set내부에 들어갈 수가 없다.
그런데 결과는 2가 출력되는데 이 경우에 hashCode를 재정의해줘야만 한다.

Map Set 인터페이스를 구현한 클래스들은 해싱을 이용해 두 객체의 Identity를 검사한다. 

기본적인 검사 과정은 아래와 같다.

    hashCode()리턴값 -> 같음 -> equals()리턴값 -> 같음 -> 동등한객체

    hashCode()리턴값 -> 같음 -> equals()리턴값 -> 다름 -> 다른 객체

    hashCode()리턴값 -> 다름 -> 다른 객체
    
위에서 hashCode를 재정의를 하지않았기때문에 Object의 hashCode메서드를 그대로 호출한다.

Object의 hashCode는 고유한 주소값을 통해 해쉬값을 반환하기때문에 위에서도 다른 객체라고 판단하고 Set에 add하게된다.

그래서 해싱기법을 사용하는 자료구조를 사용할 땐 반드시 equals와 hashCode를 재정의해야한다.

hashCode 일반규약
---
1. equals가 변하지않았다면 런타임중 hashCode는 매번 같은 값을 반환해야한다.

2. equals가 같다고 판단했다면 두 객체의 hashCode는 같아야한다.

3. equals가 다르다고 판단했어도 hashCode가 다른값을 반환할 필요는 없지만 성능에 좋지않다.

최악이지만 적법한 해쉬코드구현
---
```java
@Override public int hashCode() {return 42;}
```
해당 클래스가 equals를 잘 정의했다면 해쉬코드가 모든 객체에대해 같더라도 다른 인스턴스로 판단은 할 수 있다.

하지만 수행시간이 매우매우 느려지게된다. 좋은 해시함수는 서로 다른 인스턴스엔 다른 해시코드를 반환해야한다.

전형적인 hashCode 메서드
---
```java
@Override public int hashCode() {
       int result = Short.hashCode(areaCode);
       result = 31 * result + Short.hashCode(prefix);
       result = 31 * result + Short.hashCode(lineNum);
       return result;
   }
```
1. 해당 클래스의 첫 번째 핵심필드(areaCode)를 기본박싱타입(Short,Integer 등등)의 hashCode를 호출해 저장한다.

>핵심필드란 equals비교에 사용된 필드를 말한다.

2. 필드가 배열이라면 핵심 원소를 별도의 필드처럼 다룬다. 모든 원소가 핵심이라면 Arrays.hashCode를 사용한다.

3. 이 과정을 반복해 모든 핵심필드들에대한 해쉬값을 31을 곱해 누적시킨다.

> 31이란 수는 홀수이면서 소수이기때문에 좋다고 한다. 이유는 잘 모르겠다...

한 줄 짜리 hashCode
---
```java
@Override public int hashCode() {
       return Objects.hash(lineNum, prefix, areaCode);
   }
```
이 hash함수는 임의의 개수만큼 객체를 받아 해시코드를 계산해준다. 매우 간단명료한 코드이지만 속도가 더 느리다. 내부에서 배열을 만들고 언박싱과 박싱을 통해 생성하기때문이다.

---
item # 12 : toString을 항상 재정의하라.
---
이펙티브자바를 읽으며 가장 쉽고 가장 명료하게 와닿은 아이템이다.

toString을 재정의하지않으면 단순한 해쉬코드를 반환한다. 전혀 유용한 형태의 정보가 될 수 없다. 만약 에러로그를 남길때에도 객체의 정보와 함께 에러로그를 남길 수도 없게된다.

toString을 재정의한다면 모든 필드에대한 정보를 그대로 제공해야한다. 객체의 모든 정보를 재정의하면 에러로그를 남길때 개발자가 명확하게 인지할 수 있다.

상위클래스에서 이미 알맞게 재정의를 했다면 재정의할 필요가 없겠지만 그렇지않은 경우엔 재정의를 통해 읽기좋은 유용한 정보들을 담아서 재정의하자.

---
item # 13 : clone 재정의는 주의해서 진행하라
---

책 내용을 정리하기전에 clone메서드에대해 알아보아야한다.

clone은 Object클래스에서 접근제어자가 protected로 제공되는 메서드다. protected란 같은 패키지 혹은 다른 패키지여도 상속받는 클래스라면 사용할 수 있는 접근제어자이다.

```java
public class Test {
    public static void main(String[] args) {
        Student s1 = new Student(22);
        s1.clone(); // 접근제어자 에러
        
    }
}

class Student implements Cloneable{
    int age;
    Student (int age) {
        this.age = age;
    }
//     @Override
//     public Student clone() throws CloneNotSupportedException{
//         return (Student) super.clone();
//     }
 }
```

Student클래스는 Object의 자손이기때문에 재정의를 하지 않더라도 s1.clone()을 호출하면 호출되어야하는게 아닌가 궁금증을 가졌다.

여기서 protected의 의미를 좀 더 자세하게 살펴봐야한다. 상속을 받게되면 상위클래스의 메서드를 이용할 수 있다. public접근제어자라면 Student클래스에 해당 메서드가 있을 것이고 Student의 외부클래스에서도 객체를 생성하면 당연히 사용할 수 있다.

하지만 protected는 상속받게되면 상속받은 클래스 내부에서만 사용이 가능하다. (같은 패키지라면 상속받지 않고도 사용이 가능하지만 Object는 java.lang패키지에있다.) 그렇기때문에 외부에서 인스턴스를 생성해서 사용하려면 Student클래스에서 무조건 재정의 되어있어야만 사용이 가능하다.(public으로 재정의)

**자 이제 clone()을 재정의해야만 사용할 수 있다는 점을 이해했다.**

>Clonalbe을 구현하는 이유는 이 클래스가 복제가능한 클래스라는 점을 알려주는 구분자의 역할 뿐이다. 내부는 비어있다.

책에서는 왜 clone을 조심해서 사용하라는지 알아보자
---
clone은 객체의 값을 복사한다. 말 그대로 인스턴스의 복제형을 제공해준다.

필드가 기본타입이거나 불변객체를 가지고있다면 문제될 일이 없겠지만, 배열같은 가변객체를 필드로 두고있다면 이상한 일이 발생한다.

왜냐하면 배열은 배열의 주소를 저장하고 주소를 통해 접근하게되는데 이 때 clone은 객체의 주소를 복사해서 인스턴스를 복제하기때문에 원본과 복제본이 같은 배열을 참조하게되고 서로 충돌을 일으키게된다.

위와같은 이유때문에 clone을 조심해서 사용해야하는 것이다.

해결방법은 간단하다. 배열에대해서 따로 clone을 호출해서 저장해주면 된다. 즉 값을 복사해서 새로운 배열을 만들어주면 된다는 뜻이다.

하지만... 책에서는 또 다른 문제의식을 던진다.

**배열은 따로 값을 복사해서 새롭게 생성해주면되는데 HashTable같은 애들은 어떻게 할래?**

hashtable의 내부는 배열의 인덱스에 링크드리스트가 연결되어있는 구조다.

책이 제시한 문제의식은 재귀적으로 복사를 통해 복제본을 만든다하더라도 링크드리스트는 같은 주소를 참조하게되므로 결국 같은 링크드리스트를 참조한다는 이야기다.

이 경우에도 해결방법은 간단하다. 링크드리스트를 깊은 복사를 통해 하나하나 복사해주면된다. HashTable.Entry 메서드를 사용해도 깊은 복사를 통해 해결할 수 있다.

정리하자면 clone메서드는 얕은 복사 깊은 복사와 관련이 있기때문에 배열을 사용한다면 주의해서 재정의해야한다. 이렇게 복잡한 과정을 통해 모든 복사를 clone에게 맡기지말고 배열이 아닌 경우엔 생성자or팩터리를 통해 복사하자.


---
item # 14 : Comparable을 구현할지 고려하라
---
Comparable은 compareTo라는 단 하나의 메서드만 정의하고있는 "인터페이스"이다.

Object의 equals와 비슷한 성격이지만 순서까지 비교할 수 있는 메서드를 재정의할 수 있다.

Comparable의 구현으로 이 인터페이스를 구현한 많은 컬렉션과 알고리즘의 힘을 누릴 수 있다.

>comparator의 compare메서드도 순서를 비교하는 메서드를 정의한 인터페이스다. 이 둘의 차이점은 간단하다. compareTo는 자기 자신과 다른 인스턴스를 비교하고 compare은 비교대상이 다른 두 인스턴스이다.

기본적인 규약은 equals와 매우 비슷하다. equals의 반사성 대칭성 추이성을 만족해야한다.

**첫 번째 : 두 객체의 순서를 바꿔도 예상한 결과가 나와야한다**

a가b보다 크다면 b는 a보다 작아야한다는 당연한 예상이다.

**두 번째 : 서로 크기가 같다면 순서를 바꿔도 크기가 같아야한다.**

a와 b가 같다면 b와 a의 크기도 같아야한다.

**세 번째 : a가 b보다 크고 b가 c보다 크다면 a는 c보다 커야한다**

또한 equals와 마찬가지로 기존 클래스에서 확장한 클래스(필드를 추가)라면 compareTo 구현이 불가능하다.

마지막으로 compareTo로 수행한 동치성 테스트(크기비교를 같다고 할 경우)는 equals의 결과와 정확히 일치해야한다.
```java
new BigDecimal("1.0");
new BigDecimal("1.00");
```
BigDecimal은 equals로 비교하기때문에 hashset은 원소를 2개 갖게된다.

하지만 treeset을 사용하면 compareTo를 사용하기때문에 같은 인스턴스를 반환해 원소를 1개만 갖게된다.

compareTo의 작성요령은 간단하다.

```java
//객체참조 필드가 하나인 비교자
public final class CaseInsensitiveString
        implements Comparable<CaseInsensitiveString> {
            public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
        }
```

위 코드를 보면 타입을 CaseInsensitiveString으로 제한했다. 그 외의 객체는 비교하지 않겠다는 뜻이다.

원래는 >, <를 이용해 비교했지만 java 7부터 정적메서드 compare로 비교하는 것을 책은 권장하고있다.

위 예시는 객체참조 필드가 하나이지만 여러개라면 어떻게 해야할까?

학생클래스의 필드에 나이, 학번, 학점이 있다면 어느 것을 먼저 비교해야할지가 중요해진다.

가장 중요한 필드부터 비교해나간다!

비교결과가 결정된다면 아래처럼 (0이 아니라면) 곧장 결과를 반환하고 0이라면 다음 필드를 비교하고 그 다음 필드를 비교하고 계속해서 이어나가면된다.

```java
//기본 타입 필드가 여럿일 때
public int compareTo(PhoneNumber pn) {
       int result = Short.compare(areaCode, pn.areaCode);
       if (result == 0)  {
           result = Short.compare(prefix, pn.prefix);
           if (result == 0)
               result = Short.compare(lineNum, pn.lineNum);
       }
       return result;
   }
```

```java
//비교자 생성 메서드를 활용
    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);
```
Comparator인터페이스에 comparingInt를 사용해 메서드체이닝방식으로 비교자를 계속해서 생성할 수 있다.

내부적으로 키를 매핑해 순서를 정하는 알고리즘을 가지고있다.

이 메서드를 이용해 CompareTo메서드를 작성할 수 있다.

```java
public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
```

comparingInt외에도 다른 타입을 지원한다. 또한 객체 참조용 비교자도 준비되어있어 이를 활용해 compareTo를 작성할 수 있다.

하지만 명심해야할 점은 약간의 성능저하가 뒤따른다.

정리하자면 순서를 고려해야한다면 ? compareable 인터페이스를 구현하자. 작성 시에 > < 연산자는 사용하지말고 기본타입클래스에서 정적compare메서드나 Comparator인터페이스가 제공하는 메서드를 사용해 작성하자.

```java
public class Test {
    public static void main(String[] args) {
        Student s1 = new Student(20,4.1,"song");
        Student s2 = new Student(24,3.2,"hong");
        Student s3 = new Student(22,4.4,"kim");
        Student s4 = new Student(26,2.4,"park");

        List<Student> list = new ArrayList<>();
        list.add(s1);
        list.add(s2);
        list.add(s3);
        list.add(s4);
        Collections.sort(list);
        System.out.println(list);
    }
}

class Student implements Comparable<Student>{
    int age;
    double gpa;
    String name;

    public Student(int age, double gpa, String name) {
        this.age = age;
        this.gpa = gpa;
        this.name = name;
    }

    private static final Comparator<Student> COMPARATOR =
            comparing(((Student s)->s.gpa))
                    .thenComparingDouble(s -> s.age)
                    .thenComparing(s -> s.name);

    @Override
    public int compareTo(Student o) {
        return COMPARATOR.compare(o,this);
    }

    @Override
    public String toString() {
        return "Student{" +
                ", gpa=" + gpa +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }
}
```

실제로 활용해본 코드이다. 정적 메서드로 COMPARATOR를 구현했고 람다식으로(익명객체) 생성했다. 내부 comparing메서드는 메서드체이닝으로 첫 번째를 비교하고 그 다음 필드를 비교하는 방식으로 작성했다. 가장 중요하다고 생각되는 필드부터(gpa) 비교를 해주었다.

왜 comparalbe을 구현해야할까? comparator만 구현하면 안되는걸까?

```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
        list.sort(null);
    }
```
Collections 클래스 내부에 선언된 sort메서드인데 이 메서드의 제네릭타입을 보면 Comparable을 이용해 정렬해준다. 정렬기준을 주지않은 클래스는 기본정렬기준을 사용한다. Student클래스는 comparable를 구현했기때문에 해당 기준을 사용해 정렬해준다.





















