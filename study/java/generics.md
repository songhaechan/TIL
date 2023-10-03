# 지네릭스 (Generics)

장점 : 컴파일 시점에 타입이 결정되기때문에 타입 안정성이 높아진다 + 타입체크 형변환을 생략하기때문에 코드가 간결해진다.

> 직접 타입체크, 형변환을 안할 뿐이지 내부적으로 타입체크와 형변환이 이루어진다.

```java
class Box<T>{}

Box<Fruit> appleBox = new Box<Fruit> // ok

Box<Fruit> appleBox = new Box<Apple> // error
```

> Apple이 Fruit을 상속받았다고 하더라도 두 번째 코드는 타입 불일치이기 때문에 error다.


### 만약 상속받은 객체끼리 할당해주고싶다면 T extends Fruit을 사용하면 가능하다.
```java

class Box<T extends Fruit>{}

class Apple extends Fruit {}

Box<Fruit> appleBox = new Box<Fruit> // ok

Box<Fruit> appleBox = new Box<Apple> // ok

```

### 지네릭 클래스 자체가 상속관계라면 아래는 가능하다.

```java
// Box가 FruitBox의 자식이라면 가능
Box<Apple> appleBox = new FruitBox<Apple>
```

### 메서드의 매개변수로도 (상속관계라면) 인자를 넘길 수 있다.

```java

Box<Fruit> fruitBox = new Box<Fruit>();

fruitBox.add(new Fruit());
fruitBox.add(new Apple()); // ok 

```

### K extends T 와 ? extends T 의 차이점

K extends T : T와 T의 자손들을 K라는 타입으로 정확히 지정하여 사용

? extends T : T와 T의 자손들의 타입을 받는데 정확히 그 자손이 누구인지는 알 수 없을 때 사용

얼핏보면 비슷하지만 사용처가 다르다.

자바에서는 ? extends T는 클래스의 선언부에 쓸 수는 없다.

와일드 카드는 매게변수를 제한하는 용도로 사용하기때문이다.

> super는 extends의 정반대 역할을 한다. K super T는 T와 T의 조상을 의미한다.

### static과 제네릭

```java
public class Student<T> {
  
    static T getName(T name) {   
        return name;
    }
}
```

위 코드는 컴파일 시 에러가 발생한다.

제네릭 타입 T는 객체 생성시 결정된다.

하지만 static은 객체 생성과는 무관하기때문에 타입이 미리 결정돼 있어야한다.

```java
public class Student<T> {
    
    static <T> T getOneStudent(T id) {
        return id;
    }
}
```

하지만 제네릭 메서드는 위처럼 가능하다.

> 클래스 레벨에 붙은 T와 제네릭 메서드에 붙은 T는 완전히 다르며 제네릭 메서드의 T는 호출 시점에 정해질 수 있다.

즉 static 제네릭 메서드의 타입 T는 객체 생성 시점이 아닌 호출 시점에 결정되기 때문에 가능하다.

### static 과 와일드 카드

```java
static Juice makeJuice(FruitBox<? extends Fruit> box){
    String tmp = "";
    for(Fruit f : box.getList()) tmp += f + " ";
    return new Juice(tmp);
}
```

위 ?에 T를 쓸 수 없다는 건 바로 위에서 알아봤다.

위에서 ?대신 T를 사용하게되면 클래스에 선언된 T와 같은 T가 되는데 클래스는 인스턴스 생성이 되지않았기때문에 T가 아직 미확정이다.

대신 ?를 사용해주면 클래스의 제네릭 타입 T와는 무관한 그저 타입을 걸러주기만하는 용도이다.

즉 타입을 클래스의 타입과 무관하게 거름망 역할만 해주게된다.

위 방식이 마음에 들지않으면 **제네릭 메서드**를 사용하면 된다.

```java
static <T extends Fruit> Juice makeJuice(FruitBox<T> box){
    String tmp = "";
    for(Fruit f : box.getList()) tmp += f + " ";
    return new Juice(tmp);
}
```

이 T는 클래스의 T와 무관한 T이기에 static메서드 호출 시점에 사용이 가능하다.

## 익명 클래스

```java
// 부모 클래스
class Animal {
    public String bark() {
        return "동물이 웁니다";
    }
}

public class Main {
    public static void main(String[] args) {
        // 익명 클래스 : 클래스 정의와 객체화를 동시에. 일회성으로 사용
        Animal dog = new Animal() {
        	@Override
            public String bark() {
                return "개가 짖습니다";
            }
        }; // 단 익명 클래스는 끝에 세미콜론을 반드시 붙여 주어야 한다.
        	
        // 익명 클래스 객체 사용
        dog.bark();
    }
}
```

만약 위처럼 익명클래스를 사용하는게 아니라면 아래와 같이 코드를 짜야한다.

```java

class Dog extends Animal{
    @Override
    publix String bark(){
        return "개가 짖습니다";
    }
}

```

Main에서 위 객체를 생성해서 호출해주어야한다.

위 객체가 정말 일회성으로 사용되는 객체라면 코드 낭비이고 메모리 낭비다.

이런 문제점을 해결하기위해 익명 클래스는 객체의 생성과 정의를 동시에하고 딱 한 번만 사용되고 사라진다.

> 오버라이딩 외에도 다른 메서드를 사용할 수 있다. 하지만 외부에서 사용은 불가능.

> 사실상 부모를 상속받는 자식클래스인데 이름이 없고 생성과 정의를 동시에 할 뿐이다.

### 일회성 클래스 외에는 다른 특징

부모를 상속받는 자식 클래스라는 것은 부모가 인터페이스여도 가능하다는 말과 같다.

즉 일회용 구현체를 만들 수 있다는 특징이 있다.

> 사용법은 똑같음.

### 어디에 사용하나?

Arrays.sort() 메서드를 봐보자.

```java
public static <T> void sort(T[] a, Comparator<? super T> c) {
        if (c == null) {
            sort(a);
        } else {
            if (LegacyMergeSort.userRequested)
                legacyMergeSort(a, c);
            else
                TimSort.sort(a, 0, a.length, c, null, 0, 0);
        }
    }
```

내부에서 Comparator를 받고있다.

> Comparator 인터페이스이고 메서드는 단 하나.

여기서 진가를 발휘한다.

위에서 익명객체는 인터페이스타입에도 구현체를 만들어줄 수 있다고 했다.

```java
Arrays.sort(users, new Comparator<User>() {
            @Override
            public int compare(User u1, User u2) {
                return Integer.compare(u1.age, u2.age);
            }
        });
```

따로 이름을 지어주어서 구현체를 만들 필요가 없다.

> 익명객체가 없다면 이름을 지어주고 따로 객체를 생성해 주입해야함

여기서부터가 람다의 시작이다.