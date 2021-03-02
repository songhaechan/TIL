# CSS

CSS는 HTML과 더불어 웹페이지의 기본 구조를 결정하는 마크업 언어로 HTML보다는 **디자인**적인 부분을 담당하는 언어이다.
CSS에는 Layout을 짜는 Grid, Flex-box 등이 있는데, 그 속에는 굉장히 많은 Property와 Value가 존재한다. 모든 Property와 Value를 암기하려 하지말고 [MDN](https://developer.mozilla.org/en-US/), [CSS Trick](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) 을 통해 직접 찾아보고 이해하는 과정을 겪도록 노력하자.  
_(ref. Dream coder)_

## CSS Flex-box

---

> [MDN](https://developer.mozilla.org/en-US/) 의 Flex-box outline을 보면, one-dimentional moduel 이라고 소개되어있다.
> 반면, CSS 의 Grid는 two-dimentional moduel 이라 한다. Flex-box는 row와 column을 제어하긴 하지만 이것이 두 축을 동시에 사용한다는 뜻이 아니다. Grid는 row와 column을 동시에 제어하는 격차축과 같다고 생각하면 된다.

Flex-box에서 가장 중요한 점은 두 가지의 축이 존재한다는 점이다.
가령 item들이 가로 배열 상태라면 가로 축이 **main axis**(주 축)가 되고, 그에 해당하는 수직 축이 **cross axis**(수직 축)이 된다. 그에 대한 역도 성립한다.

<img src=".\img\1.jpeg">
<img src=".\img\2.jpeg">

- flex-direction : 정렬할 방향을 정한다. ( row, column )
- justify-content : 가로선 상에서 정렬
- align-items : 세로선 상에서 정렬
- flex-wrap : 요소를 여러줄에 걸쳐 정렬

> justify-content 에서 space-between과 space-around를 가장 많이 사용했다. **between**은 _양 옆에 여백을 남기지않고_ 요소를 동일한 간격으로 배열하고, **around**는 *양 옆에 여백을 포함*하여 요소를 동일한 간격으로 배열한다. 이는 align-items 도 마찬가지로 적용된다.

<img src=".\img\3.jpeg">

이 외에도 여러 property가 존재하지만, 필요할 때 MDN을 참고하자.
메인 아이디어를 이해한다면, property와 value는 찾아서 해결하자.

## CSS Grid

---

웹페이지 레이아웃을 결정짓는 양대산맥은 Flex-box와 Grid 이다.
2021년 현재 flexbox는 90%넘는 이용자가 있고, gird는 60%정도가 사용한다고 한다.

Gird는 좌표축과 같이 격자를 생성해 요소를 배열한다.
Flex-box는 요소를 조금더 자유롭게 배열할 수 있지만, Grid는 격자에 맞춰 배열하게된다.

위에서 정리한 Flex-box와 마찬가지로 요소를 감싸는 부모태그가 꼭 필요하다. 이를 grid-container이라 한다.

<img src=".\img\5.jpg">

```html
<!-- html -->
<div class="grid-container">
  <div class="item1">Header</div>
  <div class="item2">Menu</div>
  <div class="item3">Main</div>
  <div class="item4">Right</div>
  <div class="item5">Footer</div>
</div>
```

커다란 grid-container 안에 각 아이템들이 배치된다.

```css
/* css */
.grid-container {
  display: grid;
  grid-template-areas:
    "header header header header header header"
    "menu main main main right right"
    "menu footer footer footer footer footer";
}
```

각 아이템을 grid-area:header 과 같이 지정하여, 원하는 면적만큼 차지하게 할 수 있다.

아이템 사이사이의 간격을 gap이라 부르며 이 값을 따로 선언하여 수정할 수 있다.

위의 코드는 **grid-template-area**를 사용했지만 **grid-template-column:auto auto auto;** 로 지정하여 열을 3개 생성할 수도 있다.

Grid의 독특한 점은 **Line**이 존재한다는 것이다.
행렬로 생각하면 쉬운데, 행과 열이 왼쪽과상단에서 Line 1 으로 시작해서 오른쪽과하단으로 증가한다.

grid-column-start (end) 로 값을 지정할 수 있다.

Grid는 격자 모양을 만들어 요소를 배치시키고싶은 면적만큼 배치시키고, 각 요소에 gap을 이용에 spacing이 가능하다.

인스타그램의 사진배열 같은 인터페이스 구성에 탁월하다.

## CSS Box-sizing

---

<img src=".\img\4.jpeg">

CSS에서 기본적으로 요소의 크기는 컨텐츠 자체의 크기다.
Hello안에 보이는 하얀 면적이 컨텐츠가 기본적으로 가지고있는 크기이고, 검은색 면적이 border영역으로 지정돼있다.
서로 다른 요소가 하얀면적을 기준으로 배열되기때문에 배열의 모양이 어긋나 보이는 경우가 많다.

이때 box-sizing:border-box 로 지정하면 border영역을 기준으로 요소들이 정렬되기때문에 서로의 컨텐츠면적에 관계없이 배열된다.

보통은 \*태그를 이용해 모든 태그에 지정한다.

> margin, padding 값도 \*태그 안에서 0으로 초기화시키는 편이 좋다.
