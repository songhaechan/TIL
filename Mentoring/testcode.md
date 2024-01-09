## 테스트 왜 해야하나?

코드를 작성하고 있는 개발자 10명에게 어떤 코드를 작성중이냐 물으면 7명은 테스트 코드를 작성 중이라고 답한다고한다.

그만큼 테스트 코드는 개발에 많은 비중을 차지한다.

### 그렇다면 왜 테스트 코드를 작성할까?

### 테스트 코드는 메서드의 문서를 대체할 수 있다.

보통 개발을하면 코드에대한 주석을 작성해 이해를 돕는다.

```java
    /**
     * Parses the string argument as a signed decimal {@code long}.
     * The characters in the string must all be decimal digits, except
     * that the first character may be an ASCII minus sign {@code '-'}
     * ({@code \u005Cu002D'}) to indicate a negative value or an
     * ASCII plus sign {@code '+'} ({@code '\u005Cu002B'}) to
     * indicate a positive value. The resulting {@code long} value is
     * returned, exactly as if the argument and the radix {@code 10}
     * were given as arguments to the {@link
     * #parseLong(java.lang.String, int)} method.
     *
     * <p>Note that neither the character {@code L}
     * ({@code '\u005Cu004C'}) nor {@code l}
     * ({@code '\u005Cu006C'}) is permitted to appear at the end
     * of the string as a type indicator, as would be permitted in
     * Java programming language source code.
     *
     * @param      s   a {@code String} containing the {@code long}
     *             representation to be parsed
     * @return     the {@code long} represented by the argument in
     *             decimal.
     * @throws     NumberFormatException  if the string does not contain a
     *             parsable {@code long}.
     */
    public static long parseLong(String s) throws NumberFormatException {
        return parseLong(s, 10);
    }
```

> parseLong 메서드에대한 주석이다.

자 여기서 parseLong 메서드를 수정해야한다면 당연히 주석 또한 수정해야한다.

즉 주석도 유지보수의 대상이 된다는 의미다.

주석을 달지 않아야한다는 것이 아니라 주석보다도 더 직관적으로 해당 메서드를 이해할 수 있는 수단이 필요하다.

---

### 오류의 위치 파악

또 개발을 하다보면 코드를 무진장 많이 작성해놓고 수정하는 경우가 많이 있다.(리팩토링)

테스트 코드가 없다면 내가 수정을 가하는 행위가 다른 곳에 어떤 영향을 주는지 알기가 매우 어렵지만, **단위 테스트(메서드레벨)** 를 작성하면 정확하게 파악이 가능하다.

### 가장 중요한 이유 : 테스트하기 좋은 코드

메서드를 테스트하려면 테스트하기 쉬워야한다.

```java
export default class Order {
    ...
    discount() {
        const now = LocalDateTime.now();
        if (now.dayOfWeek() == DayOfWeek.SUNDAY) {
            this._amount = this._amount * 0.9
        }
    }
}
```

위 코드는 테스트하기 매우 어렵다.

일요일에만 테스트해야하는 상황이 발생하기 때문이다.

### 테스트코드의 핵심은 순수함수다.

즉, 여러번 호출하더라도 항상 같은 값을 반환해야하는데 위 경우 요일에 종속적인 값이 나온다.

즉 테스트하기 어렵다.

그렇다면 이렇게 해보자

```java
export default class Order {
    ...
    // 현재시간(now)를 밖에서 주입받도록 한다.
    discountWith(now: LocalDateTimw) {
        if (now.dayOfWeek() == DayOfWeek.SUNDAY) {
            this._amount = this._amount * 0.9
        }
    }
}
```

위 코드는 테스트하기 매우 쉽다.

그리고 항상 같은 값을 반환한다.

### 테스트하기 쉬우면 좋은 코드인가?

테스트하기 쉽다고 모두 좋은 코드는 아니다.

하지만 좋은 설계, 좋은 코드는 테스트하기 쉽다.

또 테스트하기 어려운 코드는 잘못된 설계냄새를 풍긴다.

주로 테스트하기 어려운 경우는 아래와 같다.

- 코드가 너무 많은 의존성을 가지고 있어, 하나의 테스트를 진행하기위해 사전준비 작업과 사후처리가 늘어난다.
- 외부 세상에 영향을 받는 코드 (위 예시)
- 외부 세상에 영향을 주는 코드 (DB 작업, 사용자의 요청 등)

테스트가 쉽다는 의미는 위 세 가지 악영향을 가지지 않는 원본 코드라 할 수 있다.

> 소프트웨어 특성은 외부 환경과의 끊임없는 소통이라 당연히 완전한 순수함수만으로 구성할 수는 없다. 하지만 이 범위를 최소화해야한다.

### 그렇다면 쉬운 테스트를 위해 원 설계를 바꿔도 되는가?

당연히 YES다. 많은 개발자들이 동의한다.

테스트 코드는 원본 코드의 부차적인 체크 수단이 아니라 같은 레벨로 봐야한다. (이동욱님 말씀)

## 외부세상에 영향을 받는 코드

우선 예시를 보기전에 간단하게 Layer 패턴을 설명하자면

Controller : 서버의 가장 앞단이다.(Interceptor나 Filer를 제외하고) 사용자의 요청을 받는다.

Service : 핵심 서비스 로직을 담는다.

Repository : 외부세상(DB)에 접근해 영속화(저장)하는 계층이다.

Domain : 단적인 예로 주문, 돈, 회원, 상품 같은 특정 범주로 묶을 수 있는 단위다.(가장 깊고 핵심이 되는 객체다.)

> Domain은 Layer라고 보긴 어려울 것 같긴하다.

```java
export default class Order {
    ...
    discount() {
        const now = LocalDateTime.now();
        if (now.dayOfWeek() == DayOfWeek.SUNDAY) {
            this._amount = this._amount * 0.9
        }
    }
}
```

위에서 보던 Order라는 Domain이다.

discount는 순수함수가 아니다.

계속되는 호출에도 같은 값을 반환하지않는다.

심지어 테스트할 때는 일요일에만 원하는 결과를 얻을 수 있다.(테스트의 할인의 성공이라면)

이에 대한 해결은 간단하다.

### 바깥쪽으로 밀어내자 (호출부로 밀어내자)

```java
export default class Order {
    ...
    // 현재시간(now)를 밖에서 주입받도록 한다.
    discountWith(now: LocalDateTimw) {
        if (now.dayOfWeek() == DayOfWeek.SUNDAY) {
            this._amount = this._amount * 0.9
        }
    }
}
```

일단 외부 의존성을 바깥에서 받도록하고

바깥에서 주입해주도록한다.

```java
export class OrderService {
    constructor(
        private readonly orderRepository: OrderRepository,
        ...
    ) {}

    async discountWith(orderId: number) {
        const order:Order = await this.orderRepository.findById(orderId);
        order.discountWith(시간);
        await this.orderRepository.save(order);
    }
    ...
}
```

### 이제 Domain 로직은 쉽다!! 그런데.. Service는?;;

서비스 레이어는 아직도 테스트가 어렵다.

그래서 외부 의존성을 최대한 정말 가능한 최대한 바깥쪽으로 밀어내야한다. (Controller까지)

즉, Controller에서 날짜를 생성해 넘겨준다.

### 그럼 Controller는?

맞다. Controller는 여전히 테스트가 어렵다.

하지만 가장 바깥으로 밀어냄으로써 적어도 서비스단과 도메인단은 테스트가 쉬워져 테스트하기 어려운 범위를 **최소화**했다.

### Controller는 Test Double을 이용해 해결하자

아주 객체지향적인 해결방법인데,

Test Double은 말그대로 테스트를 위해 인터페이스를 두고 두 구현체를 구현해 하나는 테스트용 하나는 구현용을 만들어 해결한다.

## 외부에 영향을 주는 코드

```java
export default class Order {
    ...
    async cancel(cancelTime): void {
        if(this._orderDateTime >= cancelTime) {
            throw new Error('주문 시간이 주문 취소 시간보다 늦을 수 없습니다.');
        }
        const cancelOrder = new Order();
        cancelOrder._amount = this._amount * -1;
        cancelOrder._status = OrderStatus.CANCEL;
        cancelOrder._orderDateTime = cancelTime;
        cancelOrder._description = this._description;
        cancelOrder._parentId = this._id;

        await getConnection()
          .getRepository(Order)
          .save(cancelOrder);
    }
}
```

위 코드는 주문을 취소하면 취소주문을 생성해 데이터베이스에 적재하는 코드다.

위 코드를 테스트하려면 어떻게 해야할까?

### 테스트 데이터 준비

일단 주문을 생성하고 상태를 취소한 뒤 디비에 잘 적재돼있는지 꺼내서 확인해야한다.

DB에 접근하는 코드는 테스트코드를 매우 느리고 만들고 사전작업 사후처리를 매우 복잡하게 만들기때문에

테스트하기 어려운 코드다.

### 외부세상과 분리

이 경우도 마찬가지로 DB접근부분은 Service레이어에 맡기고 도메인 로직은 그저 데이터를 생성해 반환하는 메서드로 변경하면 된다.

### 그럼 Service 레이어는?

어쩔 수 없다. 어차피 DB접근은 해야만한다. 단지 테스트하기 좋은 범위를 확장시키고 테스트하기 어려운 범위를 최소화했을 뿐이다.

## 정리

정말 너무 단순화해서 글을 썼다.

테스트 코드에 관심이 있다면...

https://jojoldu.tistory.com/674?category=1036934

참고해보면 좋을 것 같다.
