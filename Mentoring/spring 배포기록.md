## EC2에 바로 배포

EC2 인스턴스를 생성하고 EC2 서버에 접속해 git clone으로 레포지토리를 가져오고 jar파일을 바로 실행시키는 것이 아주 간단한 배포방법이다.

EC2의 공인아이피 주소를 탄력적 IP로 생성했기때문에 서버를 재가동해도 IP는 움직이지 않도록 설정했다.

처음엔 http://54.xxx.xxx.xx로 접속했다.

당연히 접속이 안된다.

Spring Boot의 WAS 톰캣이 8080포트를 열어놓고 Listen 중이기 때문이다.

http://54.xxx.xxx.xx:8080으로 접속해야한다.

> Spring yml 파일을 설정하면 default(8080)값이 아닌 다른 포트를 열어놓을 수는 있다.

이 방식엔 자동으로 배포가 되지않는다는 크나큰 문제가 있다.

깃헙에 push하면 그때마다 ec2에 접속해서 다시 git pull을 통해 레포지토리를 갱신하고 다시 jar파일을 실행해야한다.

서버가 100개면 100번 저 작업을 해야한다.

심지어 ec2에서 빌드도 따로 해야한다.

## GitHub Action을 사용하자

CI/CD 툴인 Github Action을 통해서 배포 자동화를 하기로 결정했다.

Github Action이 해주는 일은 아래와 같다.

- jdk 세팅
- 테스트 및 빌드
- 빌드 결과물 압축
- 원격으로 AWS에 접속
- S3에 압축파일 전송
- CodeDeploy가 압축된 파일을 압축해제 후 EC2에 배포
- 배포한 jar파일을 실행

이 모든 작업은 메인 브랜치에 push 하거나 pr merge 시 자동으로 이루어진다.

## Nginx를 이용해 Reverse Proxy 서버를 사용해보자.

Nginx는 WAS서버 앞단에서 가장 먼저 요청을 받는다.

정적작업은 (html,js,css)는 Nginx가 처리하고 동적작업들은 WAS에 위임해서 결과를 대신 Nginx가 내려준다.

또 다른 특징은 nginx가 모든 요청을 앞단에서 받기때문에 클라이언트는 직접 WAS서버와 통신하지않기 때문에 보안 측면에서도 얻는 이점이 있다.

nginx가 요청을 가장 앞에서 받기때문에 L7 로드밸런싱 기능을 사용할 수 있다.

nginx를 설치하니 http://54.xxx.xxx.xx:8080으로 접속하지 않아도 됐다.

포트를 생략해도 접속이되는데 왜 그럴까?

### Nginx의 nginx.conf 파일을 보면 80포트를 listen하고있다.

http의 기본포트가 80이기때문에 생략가능하고 80을 nginx가 듣고있기때문에 가능한 일이다.

외부에서 80포트로 들어온 요청을 nginx가 was의 8080으로 넘겨주게된다.

## Https 프로토콜을 사용하자

Nginx에 ssl 인증서를 적용해서 https 통신을 할 수 있다.

기존에 http의 요청은 모두 https로 리다이렉션하기로 했다.

```shell
server {
    if ($host = haechan.store) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


	listen 80;

    server_name haechan.store www.haechan.store;
    return 404; # managed by Certbot
}
```

위 .conf파일의 server블록은 Certbot이 작성한 블록이다.

80을 Listen중이고(http요청을 listen) 모든 요청은 https로 301 리다이렉션한다.

```shell
server { # server 블록

    server_name haechan.store www.haechan.store;

    access_log /var/log/nginx/proxy/access.log;
    error_log /var/log/nginx/proxy/error.log;

    location = /favicon.ico {
    return 204;
     access_log     off;
    log_not_found  off;
}
    location / { # location 블록
        include /etc/nginx/proxy_params;
        proxy_pass http://54.xxx.xxx.xx:8080;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/haechan.store/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/haechan.store/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
```

위는 기본적으로 443포트를 listen중이고 https로 들어오는 요청을 받는다.

그 후에 api서버(tomcat)로 요청을 전달한다.

> 질문
>
> 1. location /{} 블록을 보면 proxy_pass를 http방식으로 넘기고 있는데 nginx와 spring 사이의 통신은 http를 사용해도 괜찮은가? 아니 애초에 패킷을 https로 보내기는 하는건가?
> 
> 2. 사용자의 요청은 http 혹은 https일텐데 http인 경우는 https로 리다이렉션하고 https는 그대로 api서버에 요청을 하는데 요청 주소가 왜 http://xx.xxx.xxx.xx:8080으로 루트패스로 보는가? https의 패킷을 변형해서 http로 변환하는 건가? 그렇다고해도 정확한 location이 루트패스가 아닐경우에도 왜 전부 루트패스로 보내는건가?

## Docker를 사용해보자.

### 우선 도커가 있기 전의 VM 기술에대해 간단히 알아보자.

VM이 등장하기전엔 1개의 서버엔 1개의 운영체제를 올려 사용했다.

하지만 하나의 애플리케이션이 사용하는 리소스가 그리 많지 않다면 하드웨어를 효율적으로 사용한다고 할 수 없다.

그렇다고 1개의 서버에 서로 다른 애플리케이션을 무작정 설치하자기엔 문제가 있는데

서로 같은 OS에서 같은 리소스를 공유하며 사용하고있기때문에 호스트 OS에 문제가 생기면 올라와있는 두 개의 어플리케이션 모두 문제가 생긴다.

### 그래서 VM이 등장했다.

Hypervisor라는 레이어를 통해서 호스트OS의 리소스를 완전히 독립적으로 격리시켜 Guest OS를 올려서 사용한다.

즉 하드웨어는 하나지만 위에 올라간 OS는 여러개가 될 수 있는 것이다.

guest OS 자체가 격리된 상태이기때문에 하나의 애플리케이션이 문제를 일으키더라도 바로 옆 guest OS는 영향을 받지 않을 수 있다.

### 하지만 VM도 문제가 있다.

하이퍼바이저를 이용한 독립적인 하드웨어 리소스 할당은 매우 무거운 작업이다.

하나의 하드웨어엔 하나의 OS가 설치되는데 하나의 하드웨어에 여러 OS가 설치된다면 리소스 부담이 매우 크다.

### Docker의 등장

"한 서버의 어플리케이션을 다른 서버로 이동시키는 것보다 지구반대편에서 자동차를 배송하는 것이 더 쉽다."

그만큼 예전 가상화를 사용하던 시절은 배포가 쉽지않았다.

무겁다는 것은 그렇다쳐도 서버가 수천대쯤 되면 전체적인 관리가 어려웠다.

**VM으로 운영체제를 가상화하기보단 애플리케이션과 그 종속성만 가상화하자는 것이 도커의 방식이다.**

컨테이너끼리는 독립된 파일 시스템(루트가 서로 다름)가지고 리소스를 서로 제한하여 하나의 컨테이너가 과도한 리소스를 소모하지 않도록한다.

또한 독립된 네트워크 인터페이스를 갖는다.

### 서버 확장에 유리

깃헙 액션과 도커를 사용하며 편리하다고 생각한 것은 

만약 다른 EC2서버를 생성한다면 그 서버에 여러 환경설정을 해야하지만 도커를 사용하면 도커허브에서 빌드된 이미지만 가져와서 컨테이너를 구동시키면 된다는 것이다.

이미 이미지에 어플리케이션 구동에 필요한 것들이 모두 설정돼있기때문이다.

## EC2 + Docker + Nginx + Spring Boot

최종적으로 위와같은 구조를 가진 프로젝트를 구성했다.

### Nginx를 이용한 로드밸런싱

내 목표는 하나의 EC2서버에 Spring 서버를 2대 띄우고 Nginx로 L7(애플리케이션 레벨)의 로드밸런싱을 적용하는 것이다.

정확히는 라운드 로빈방식으로 순차적으로 트래픽을 분산시키려는 목적이다.

nginx의 설정으로 ip_hash, 가중치부여 등 다른 여타 알고리즘이 있지만 가장 기본적인 설정을 사용하기로했다.

> 질문
> 
> nginx까지 컨테이너로 띄울 필요가 있을까?

