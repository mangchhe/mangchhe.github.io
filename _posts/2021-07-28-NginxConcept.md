---
title: Nginx? 왜 쓰는지? 설치와 사용법에 대해서 알아보자
decription:
categories:
 - WEB
tags:
 - was
 - load balancer
 - proxy
 - nginx
 - apache
---

> Nginx를 왜 쓰는지, Apache 서버와 차이점은 무엇인지, Nginx 설치하는 방법과 사용법에 대해서 알아보자

> ## Nginx를 쓰게된 이유?

AWS EC2에 진행중인 프로젝트를 배포하려고 인스턴스를 생성하는 도중에 생각이 들었다. 이전과 프로젝트 배포들과 동일하게 빌드하고 jar파일 만들어 서비스 하나를 구동시키기에는 내가 이전과 다르게 좀 많은 것을 알아버렸다는 것을 깨달았다.

사실 실제 배포할 생각으로 이 프로젝트를 진행한 것이 아니라 트래픽도 많이 생길 염려도 없지만 로드 밸런싱도 해보고 무중단 배포해보는 것도 재미있을 것같고 경험이 될꺼 같아 해보고싶었다. 그래서 고민을 하다가 이전에 배운 spring cloud gateway를 이용해서 해볼까하다가 새로운 것을 접해보고 싶어서 다른 것을 찾아보게 되었고 elb(elastic load balacing)과 nginx를 찾게 되었고 aws에 너무 의존하는 느낌이 들어서 프록시 서버로 nginx를 사용해보기로 하였다.

> ## Nginx가 무엇일까?

Nginx는 Apache와 같은 웹 서버이며, 가벼움과 높은 성능을 목표로 한다고한다.

웹서버이기에 클라이언트에게 정적 데이터를 전달하기 위해 사용이 되기도하고 비동기 Event-Driven 방식으로 reveser proxy server로 이용되 로드밸런싱을 사용하기도 한다.

> ## 그럼 Apache도 있고 Nginx도 있는데 왜 Nginx를 사용할까?

이 둘의 차이점은 Apache는 **쓰레드 / 프로세스 기반 구조**로 요청 하나당 쓰레드 하나가 처리하는 구조로 Spring Boot를 생각할 수 있고 Nginx는 **비동기 이벤트 기반 구조**로 NodeJs를 생각할 수 있다.

![threadeventdriven](/assets/postImages/NginxConcept/threadeventdriven.JPG)

Apache는 I/O 요청을 하고 응답이 올때까지 아무것도 하지 않고 시간을 낭비하는 쓰레드 지연문제가 발생하고 컨텍스트 스위칭이 발생하기 때문에 오버헤드가 발생하는 반면 비동기 이벤트 기반 구조는 I/O 요청이 발생하면 응답을 기다리지 않고 다음 작업을 실행해버리고 요청은 이벤트 핸들러에 등록되어 비동기 방식으로 처리되어 로직이 진행되며 끝날시에 콜백함수를 부르는 방식으로 Apache에 비해 적은 쓰레드로 많은 양의 트래픽을 처리할 수 있다.

**즉, 제일 앞단에서 들어오게 되는 모든 요청들을 받아내 요청을 분배하기 위해서는 Apache 보다는 Nginx가 Proxy Server에 적합하다.**

> ## 설치 방법

설치 방법부터 실습까지 환경은 **Ubuntu 18.04 LTS**에서 진행된다.

``` bash
sudo apt update
sudo apt install nginx

sudo apt service nginx status # 구동확인
sudo apt service nginx start/stop/restart # 시작, 정지, 재시작
```

> ## 실습

![nginxarchitecture](/assets/postImages/NginxConcept/nginxarchitecture.JPG)

실습 내용은 간단한 node 서버 예제를 이용하여 8080, 8081, 8082 포트를 가진 서버 세개를 띄워 로드밸런싱이 잘 이루어지는지 확인하는 것으로 진행된다.

> #### 설정 파일 생성

``` bash
cd /etc/nginx/site-enabled # 설정 파일 저장 위치
mkdir <파일명>.conf # 설정 파일 생성
```

> #### 설정 파일 내용

``` nginx
server {
        listen 80;
        listen [::]80;

        server_name <호스트 네임>;

        location / {
                proxy_pass http://node-app;
        }
}

upstream node-app {
        # round-robin(default) least_conn, ip-hash 로드밸런싱 방식 설정
        server 127.0.0.1:8080
        server 127.0.0.1:8081
        server 127.0.0.1:8082
}
```

로드 밸런싱 종류 참고 -> [링크](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)

> #### 설정 파일 적용

``` bash
sudo service nginx reload/restart
```

> #### nodeJs 간단 웹 예제

``` js
const http = require('http');
const port = 8080; // 8081, 8082
http.createServer(function(request, response){
  console.log('HTTP server start on port ' + port);

  response.writeHead(200, {'Content-type': 'text/plan'});
  response.write('Hello Node World!');

  response.end( );
}).listen(port);
```

> #### node 서버 실행

``` bash
node <파일명>.js & # port : 8080
node <파일명>.js & # port : 8081
node <파일명>.js & # port : 8082
netstat -ntlp | grep <파일명> # 정상 실행 확인
```

![nodePort3](/assets/postImages/NginxConcept/nodePort3.JPG)

> #### 요청 보내기

![nginxloadbalance](/assets/postImages/NginxConcept/nginxloadbalance.JPG)

각 서비스에 요청이 골고루 전달되는 것을 확인할 수 있고 로드 밸런싱 방식은 디폴트인 라운드 로빈 방식으로 설정되어 있어 순차적으로 돌아가며 요청이 전달되게 된다.

> ## Reference

- [https://www.iij.ad.jp/en/dev/tech/mighttpd/](https://www.iij.ad.jp/en/dev/tech/mighttpd/)
