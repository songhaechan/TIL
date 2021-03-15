# Node.js

## URL

URL은 접속하고싶은 페이지의 주소라는 정도만 알고있었지 정확한 구조를 알지 못하고있었다.  
생활코딩에서 URL의 QueryString을 이용해 동적웹페이지를 구현하며 URL의 기본구조에대해 배웠다. protocal,auth,host 등 여러 개념이 있지만 그 중에서 **QueryStirng**에 주목해서 공부했다.

> **QueryString** : /?id=something 의 구조로 pathname 뒤에 붙는 부가적인 정보이다. 클라이언트가 정확히 어떤 리소스에 접근하고자하는 정보를 웹서버에게 전달한다. (?의 의미는 QueryString이 시작된다는 뜻이다.)

http://localhost:3000/?id=CSS 에 접속한다고 가정하자.

우선 Node.js 에서 url모듈을 가져온다.

```js
var url = require("url");
var _url = request.url; // request를 define 하지않았음.
var queryData = url.parse(_url, true).query;
console.log(queryData);
```

request.url을 통해서 현 페이지 url을 가져오고 queryData에 저장한다.  
parse를 이용해 URL주소를 객체로서 가져올수가있다.
아래는 queryData의 출력결과이다.

```js
// [Object: null prototype] { id: 'CSS' } // bash
```

이렇게 객체로서 출력되는데, queryData.id를 출력하면 CSS(value)만 얻어낼 수가 있다.

HTML href 에 QueryString으로 링크를 걸어두면 페이지를 리로드할 필요없이 접속이 가능해진다.

여기서 가장 중요한점은 우리가 주소창에 /?id=something 으로 작성하고, respond.end(queryData.id)라고 코드를 작성하면 something이 출력된다는 점이다.

즉, 클라이언트의 요구를 서버에 즉각적으로(동적으로) 반영이 가능하다는 뜻이다.

클라이언트가 CSS라는 주소를 입력한다면 변수로 설정된 모든 queryData.id에 동적으로 반영이된다.
