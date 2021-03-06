# Vanilla JS

HTML과CSS는 사진이라 한다면, JS는 동영상이라 할 수 있다.  
HTML과 CSS으로 이루어진 웹페이지에 Interative한 동작을 가능하게하고 사용자와 상호작용을 활발하게 한다.  
JS는 많은 library 와 framework들을 가지고 있지만 우선 날 것의 JS를 배워보려한다.

Nicolas의 말에 의하면, JS는 굉장히 방대하다고 한다.
자신도 JS를 다 알지 못한다고 말 할정도이니...  
JS역시 필요할 때 필요한 만큼 능동적으로 공부하자!

## Vanilla JS ?

---

js가 붙는 framework들이 얼마나 많던지, 학습하기 전에 혼란스러웠다. 무엇을 먼저 배워야할 지, 무엇이 어떤 기능을 가지고 있는지 도무지 알 수가 없었다. Nicolas의 말에 따르면 React.js Vue.js 와 같은 framework들은 JS를 좀 더 _sexy_ 하게 만들어주는 하나의 도구라고 했다. 결국 뿌리는 Vanilla JS에 두고 있고, 날 것 그대로의 JS 즉, Vanilla JS배우는것이 편리한 framework를 배우는 것보다 우리의 학습을 미래지향적으로 만들어 줄 것이다.

## Variable[변수]

---

```c
int a=10;
double b=10.5;
```

위 의 코드는 c언어에서 사용하는 변수를 선언하는 방법이다.

> Data type (Variable) = Value;  
> Data type을 정확히 명시하여 선언한 뒤 변수명을 주고, 값을 대입한다.

하지만 **JS**에서는 Data type을 명시해 줄 필요가 없다. **const, let, var** 를 이용해 선언해준 뒤 값을 대입한다.

```js
const a = 10;
let b = 20;
var c = "Vanilla JS";
```

- const : 변수를 상수화하여 값을 변경하지 못하도록한다. ( 재선언, 재할당 불가능)
- let : 선언 범위가 block 범위로 선언되고, 값이 변경가능하다. ( 재할당 가능 재선언 불가능)
- var : 선언 범위가 function 범위로 선언되고, 값이 변경가능하다. ( 재할당,재선언 가능)
  > 기본적으로 const를 사용하자 (error 방지) , 재할당이 필요한 경우 제한적으로 let을 사용하자.

## Data Type

---

C언어를 처음으로 배워서 그런지, 그 뒤에 배우는 언어들을 C와 비교해보게 된다.  
JS의 datatype은 C만큼 번거롭거나 다양하지는 않다.

```javascript
const String = []; // 문자열
const Int = 10; // 정수
const Float = 10.2; // 실수
const Boolean = true; // 참거짓
```

```javascript
// Array
const daysOfWeek = ["mon", "tue", "wed", "thu", "fri", "sat", "sun"];
const num = [1, 2, 3, 4, 5];
const float = [1.1, 2.2];
```

C언어에 비해 매우매우매우 간단하다. 따로 타입을 선언할 필요가 없다. Python과 같이 미리 할당된 박스(메모리)를 이용한다고 한다.
_(매우편하다)_

JS는 배열(Array)외에 다른 방식으로 데이터를 정리할 수 있다.
객체(Object)를 이용하면 변수를 만들어 데이터를 따로 저장할 수 있다.

```js
const Info = {
  name: "haechan", // 변수명 name , 데이터 "haechan" (string)
  age: 23,
  favFood: ["Pizza", "Chicken"], //객체 속에 배열
  favMovie: [
    { name: "Oldboy", isNetflix: false }, //배열 속에 객체
    { name: "Harry Potter", isNetflix: false },
  ],
};

console.log(Info.favFood[0]);
console.log(Info.favFood[1]);
console.log(Info.name);
console.log(Info.favMovie[1]);
console.log(Info.favMovie[0].name);
console.log(Info.favMovie[1].isNetflix);
```

객체의 이름은 Info로 지정하고 {}괄호를 열어 변수명을 하나씩 지정한다. name , age 등이 변수명이 된다.

특이한 점은 객체속에 배열을 넣을 수 있고, 배열 속에 객체를 넣을 수도 있다.

객체에 접근할 때에는 .을 이용한다.  
예를 들어, Oldboy에 접근하고싶다면 Info _안에_ favMovie[0] _안에_ name 을 찾아가면 되는데 _안에_ 대신 .을 붙여준다.
C언어 구조체에서 멤버변수를 참조할때와 비슷하다.

```js
Info.favMovie[0].name; // Oldboy
```

## DOM

---

무심코 `document.querySelector("something")` 이라는 코드를 쓰며 DOM이란 개념은 정확히 들여다보지 않았다.

DOM이란 Document Object Model 의 약자로, 번역하면 문서객체모델... 이 된다.

간단히 말하자면 CSS에서 class에 접근하기위해서는 `.something{}` 으로 사용한다.  
하지만 JS는 DOM이라는 모델을 사용해 html의 태그,id,class에 접근한다.

```js
var header = document.createElement("h2");
var textNode = document.createTextNode("Hello DOM");
header.appendChild(textNode);
```

h2태그를 생성하고 header에 저장하고, textNode에 'Hello DOM'을 저장하고 header 에 자식노드를 추가하는 코드이다.

이처럼 html에있는 class id tag를 객체로서 동적으로 할당하고 할당받는 것이 DOM이다.

더 간단히 요약하자면, JS에서 HTML tag id class를 다루게해주는 도구라고 생각하면되겠다.

## Event Bubbling

---

이벤트의 흐름에 Bubbling이라는 성질이 있는데, 정확히 왜 이런 흐름을 정해놨는지는 모르겠다.  
자식노드에서 발생한 이벤트가 부모노드에게 전달되어 이벤트가 연쇄적으로 발생하게되고, 이 흐름이 버블링하는 모습과 유사해서 이런 이름을 붙였다고 한다.

```html
<form onclick="alert('form')">
  FORM
  <div onclick="alert('div')">
    DIV
    <p onclick="alert('p')">P</p>
  </div>
</form>
```

위 코드에서 지정한 onclick이벤트들은 버블링을 이해하기 아주 좋다.  
p를 클릭하면 p를 alert해주고, 이벤트를 발생시키지도 않은 div와 form의 이벤트도 alert해주게된다.

즉 가장 안쪽의 노드(leaf)부터 시작해 상단의 요소(root)까지 이벤트가 발생하게된다.

ToDo리스트를 만들던 중에 이벤트가 연쇄적으로 처리되어 곤란했는데, 이런 bubbling을 막을 코드가 있다.

`event.stopPropagation();` 이 코드로 bubbling을 막을 수 있다.
