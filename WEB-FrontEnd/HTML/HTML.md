[HTML](#html)

- [HTML 기본적인 코드 구조](#html-기본적인-코드-구조)
- [코드 구성](#코드-구성)
- [HTML의 Box 구조](#html의-box-구조)
- [HTML 과 CSS의 Box-Model](#html-과-css의-box-model)

# HTML

HTML은 CSS JS 와 더불어 웹페이지를 구현하는 **markup** 언어이다.  
HTML markup은 요소(Elements)와 속성(Attributes) 그리고 문자 기반 데이터 형태와 문자 참조와 엔티티 참조를 포함하는 몇 가지 핵심 구성 요소로 이루어져 있다.

<p>&nbsp;</p>

## HTML 기본적인 코드 구조

---

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title></title>
  </head>
  <body></body>
</html>
```

상단의 코드가 HTML의 기본구조이다.

> `<!DOCTYPE html>`

- HTML의 버전을 웹페이지에게 알려주는 선언문으로 가장 먼저 선언
  > `<html></html>`
- 이 태그로 감싸진 부분은 html의 문법으로 해석하라는 태그
  > `<head></head>`
- 문서에대한 메타데이터의 집합을 정의할 때 사용하는 태그
  > `<body></body> `
- HTML의 텍스트, 이미지, 영상 등 모든 컨텐츠를 포함하는 태그
<p>&nbsp;</p>

## 코드 구성

---

```html
<p class="edit">This is HTML</p>
```

`<p>` : Opening tag  
 `</p>`: Closing tag  
 `<p></p>` : Element  
 `This is HTML` : Content  
 `class="edit"` : Attribute

<p>&nbsp;</p>

## HTML의 Box 구조

---

네이버,다음,Google 등 유명 포털사이트들은 사실 많은 Box 구조로 이루어져 있다.  
모든 사이트들이 모두 같은 구조를 사용하는 것은 아니지만, 공통점은 존재한다.

<img src="https://1.bp.blogspot.com/-byyR6UhzRlw/XqPR9QUH12I/AAAAAAAACf8/_h6ITaQ45h0dazPFuifNqe7OSMFNbZopgCLcBGAsYHQ/s1600/HTML%2Blayout.png" width="80%">

> 솔직히 위 구조를 div 태그나 span 태그로 모두 대체해도 사용자에게 표시되는 컨텐츠는 문제없이 표시되지만, _코드의 가독성_ 과 페이지의 _논리적 구조_ 에 큰 문제가 생기므로 위 태그를 이용해 구조를 갖춰나가자.

**HTML을 마크업 할 때 웬만해서는 의미적요소가 들어있는 태그를 사용하도록하자. 특히 div는 사용해서 안되는것은 아니지만, 절대 남발하지말자.**

## Block & Inline

---

<img src="https://media.vlpt.us/images/taeha7b/post/ecde13b6-b3f1-4142-b932-dc8e553d42e2/Screenshot%20from%202020-07-22%2008-18-14.png">

Box-Model에서 화면을 사용하는 크기에 따라 두 가지로 나뉘는 요소가 존재한다.

> Block Element : 하나의 요소가 한 줄 전체를 box로 사용하는 요소  
> Inline Element : 서로 다른 요소가 한 줄을 같이 사용하는 요소

HTML의 간단한 기준이 있다면 Block요소와 Inline요소를 잘 구분해서 사용해야한다.

각 요소는 자기자신을 포함할 수 있다. Block은 Block을 Inline은 Inline을... 또한 Block은 Inline을 포함 할 수 있다. **하지만 Inline은 Block을 포함해서는 절대 안된다**

> span>div, span>h1 금지
