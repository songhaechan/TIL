# CSS
CSS는 HTML과 더불어 웹페이지의 기본 구조를 결정하는 마크업 언어로 HTML보다는 **디자인**적인 부분을 담당하는 언어이다.
CSS에는 Layout을 짜는 Grid, Flex-box 등이 있는데, 그 속에는 굉장히 많은 Property와 Value가 존재한다. 모든 Property와 Value를 암기하려 하지말고 [MDN](https://developer.mozilla.org/en-US/), [CSS Trick](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) 을 통해 직접 찾아보고 이해하는 과정을 겪도록 노력하자.   
*(ref. Dream coder)*
## CSS Flex-box
___
>[MDN](https://developer.mozilla.org/en-US/) 의 Flex-box outline을 보면, one-dimentional moduel 이라고 소개되어있다.
반면, CSS 의 Grid는 two-dimentional moduel 이라 한다. Flex-box는 row와 column을 제어하긴 하지만 이것이 두 축을 동시에 사용한다는 뜻이 아니다. Grid는 row와 column을 동시에 제어하는 격차축과 같다고 생각하면 된다.   

Flex-box에서 가장 중요한 점은 두 가지의 축이 존재한다는 점이다.
가령 item들이 가로 배열 상태라면 가로 축이 main axis(주 축)가 되고, 그에 해당하는 수직 축이 cross axis(수직 축)이 된다. 그에 대한 역도 성립한다.   
<img src="/CSS/img/1.png">
