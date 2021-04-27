# Node.js

## url 모듈

URL은 접속하고싶은 페이지의 주소라는 정도만 알고있었지 정확한 구조를 알지 못하고있었다.  
생활코딩에서 URL의 QueryString을 이용해 동적웹페이지를 구현하며 URL의 기본구조에대해 배웠다. protocal,auth,host 등 여러 개념이 있지만 그 중에서 **QueryStirng**에 주목해서 공부했다.

> **QueryString** : /?id=something 의 구조로 pathname 뒤에 붙는 부가적인 정보이다. 클라이언트가 정확히 어떤 리소스에 접근하고자하는 정보를 웹서버에게 전달한다. (?의 의미는 QueryString이 시작된다는 뜻이다.)

http://localhost:3000/?id=CSS 에 접속한다고 가정하자.

우선 Node.js 에서 url모듈을 가져온다.

```js
var url = require("url");
var _url = request.url; // request를 define 하지않았음.
var queryData = url.parse(_url, true).query;
// parse의 true는 url객체의 query 속성을 객체로 가져온다.
// parse의 false는 url 객체의 query 속성을 문자열형식으로 가져온다.
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

---

## fs 모듈 [readFile]

`var fs = require("fs");` fs라는 변수에 nodejs에서 제공하는 fs모듈을 할당할 수 있는데, fs는 filestream의 약자로 외부 파일에서 데이터를 읽어올 수가 있다.

fs모듈을 이용한 함수는 node.js 공식홈페이지에 많이 소개되어있다.  
지금은 `fs.readFile(filename, [options], callback)` readFile을 이용해 .txt 파일을 읽어와볼것이다.

위에서 모듈을 할당하고 위에 보이는 파라미터를 채워야한다.
filename은 읽어올 파일의 이름, options에는 읽어올 방식인데 주로 utf-8인코딩방식을 사용한다.

그 후에 callback에 함수를 전달한다.

```js
fs.readFile('text.txt', 'utf8', function(err, data) {
    console.log(data)
};
```

터미널에 'text.txt'를 출력하는 코드다. callback에 function(err,data)라는 함수가 전달되었고 txt에 있는 데이터가 매개변수 data로 전달되어 터미널에 .txt파일이 출력된다.

사실 위 방법은 **비동기적인** 방법을 채택한 함수이다.
**동기적 방식**은 `fs.readFileSync` 를 이용한다.

**비동기**는 파일을 읽는 중에 다른 처리가 가능하지만,
**동기**는 불가능하다.

`function(err,data)` 에서 err은 예외처리로 파일을 읽거나 쓸 때 권한거부,없는 파일 일 때 예외처리가 발생한다. 오류의 정보를 err에 전달하여 `console.log(err)` 오류를 출력한다.

---

## fs 모듈 [readdir]

```js
fs.readdir("/.data", function (err, filelist) {
  console.log(filelist);
});
```

위 코드는 fs의 readdir method를 사용해 파일 목록을 얻어오는 코드다.  
'/.data'파일의 목록을 얻어와 filelist에 전달하고 filelist를 출력한다.

```js
function templateList(filelist) {
  var list = "<ul>";
  var i = 0;
  while (i < filelist.length) {
    list = list + `<li><a href="/?id=${filelist[i]}">${filelist[i]}</a></li>`;
    i = i + 1;
  }
  list = list + "</ul>";
  return list;
}
```

filelist와 반복문을 이용해서 html파일 요청을 간결하게 만들 수 있다.

C언어를 떠올리면 아주 쉽게 이해할 수 있어서 좋았다.
변수에 문자열을 저장하기도 쉽고, 문자열을 덧붙이기에도 아주 편하다.

---

## 서버 생성

```javascript
var http = require("http");
var fs = require("fs");
var app = http.createServer(function (request, response) {
  var url = request.url;
  if (request.url == "/") {
    url = "/index.html";
  }
  if (request.url == "/favicon.ico") {
    response.writeHead(404);
    response.end();
    return;
  }
  response.writeHead(200);
  response.end(fs.readFileSync(__dirname + url));
});
app.listen(3000);
```

Node에서 제공하는 http 모듈을 가져와 **createServer method** 를 이용해 서버를 생성한다.

url,fs 모듈과 같이 http도 많은 메소드들을 가지고있다.

콜백함수에 req,res를 인자로 주고, 마지막에 `app.listen(3000)` 을 이용해 서버가 3000번 포트에서 기다리게 해준다.

포트번호 3000은 어떤 번호가 돼도 상관이없다. http://localhost:(포트번호) 형식으로 접속할 때 이용한다.

콜백 함수안에 request.url은 현재 url을 요청한다. 루트로 접속했을 경우엔 준비한 html파일을 response해준다.

루트가 아닌 /favicon.ico로 접속했을 경우엔 브라우저에 404에러를 전송해주어야 한다. 정상적인 경로로 들어오지않았기때문이다.

통상 브라우저는 응답받은 결과를 렌더링하기위해선 응답헤더를 수정해줘야하는데, 이때 404는 클라이언트 에러, 200은 성공메세지를 띄워준다.

마지막 `response.end()` 함수로 서버에게 헤더와 내용의 전송이 완료되었다는 의미와 함께 클라이언트에게 내용을 전송해주는데, fs 모듈을 이용해 폴더 내 index.html 파일로 연결시켜준다. \_\_dirname 은 현재 파일을 의미한다.

- 기본적으로 서버 인스턴스를 생성하고 createServer 의 콜백함수 인자 req,res 를 이용한 메소드들을 이용한다
