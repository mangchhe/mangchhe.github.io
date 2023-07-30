---
layout: post
title: "Hypertext Transfer Protocol(HTTP)"
categories: web
tags: web
---

인터넷 상에서 클라이언트 - 서버 간에 데이터를 주고받기 위한 프로토콜이다 HTTP는 HTML 문서, 이미지, 동영상, 등 모든 종류의 데이터를 주고 받을 수 있다

## HTTP 특징

- **비연결성(Connectionless)** : 클라이언트와 서버가 연결을 맺은 후, 통신이 끝나게 되면 연결을 끊는 것을 말한다
- **무상태(Stateless)** : 비연결성이기 때문에 서버에서는 클라이언트에 대한 이전 상태정보를 가지고 있지 않을 것을 말한다

### 장점

불특정 다수에 클라이언트들이 접속하는데 그 모든 사람들에 대해서 연결을 계속 유지해야 한다면 많은 리소스들이 발생해야하지만 유지하지 않기 때문에 **불필요한 리소스를 절약할 수 있다**

### 단점

클라이언트의 이전 상태들을 기억하고 있지 않기 때문에 매번 새로운 사람들이 들어오는 것과 같아 상태를 유지 시켜줄 수 없다

이러한 단점을 극복하기 위해서 **쿠키, 세션, JWT**와 같은 방법을 이용한다

## HTTP METHOD

클라이언트가 서버에게 요청을 할 때 어떤 목적을 가지고 있는지에 대해서 표현하는 것을 말한다

- **GET** : 요청 받은 URI의 정보를 검색하여 응답한다
- **POST** : 요청된 자원에 대해서 생성한다
- **PUT** : 요청된 자원에 대해서 수정한다
- **DELETE** : 요청된 자원에 대해서 삭제한다

CRUD는 데이터 처리의 기본 기능이고 해당 HTTP METHOD와 맞춰본다면

- POST - C(CREATE)
- GET - R(READ)
- PUT - U(UPDATE)
- DELETE - D(DELETE)

다음와 같이 매칭될 수 있음을 알 수 있다

이외에도 아래와 같은 HTTP METHOD들이 존재한다

- **HEAD** : GET과 유사하지만 BODY를 제외하고 응답코드만 전달한다
- **OPTIONS** : 웹 서버측 제공 메소드, HTTP 헤더에 담아서 전달한다
- **TRACE** : 요청 리소스가 수신되는 경로를 보여준다
- **CONNECT** : 요청 된 리소스와 양방향 통신을 시작한다
- **PATCH** : PUT과 유사하고 전체 자원을 갱신하는 반면 PATCH는 일부만 수정시 사용한다

### POST과 PUT의 차이점

- 멱등성(Idempotent)
여러 번을 수행해도 값이 같은 것을 의미한다
  - POST : 리소스를 추가하는 연산으로서 Idempotent하지 않다
  - PUT : 값을 계속 UPDATE 수행해봤자 같은 결과가 나오기 때문에 Idempotent하다
- 리소스 결정권
URI가 서버? 클라이언트? 누구에 의해 결정되는 것인지를 의미한다
  - POST : `POST /board` 다음과 같이 게시글 생성 요청을 보낼 때 생성할 때 서버에서 리소스 위치를 결정한다
  - PUT : `PUT /board/12` 다음과 같이 해당 board를 수정하기 위해서는 클라이언트에서 12와 같이 특정지어주어야 한다

## HTTP State Code

### 200번대 : 성공

- **200 : 요청이 성공적으로 완료됨**
- 201 : 요청이 성공적으로 완료되었고 그 결과 새로운 리소스가 생성됨(일반적으로 POST)
- 202 : 요청은 받아들여졌으나, 아직 동작을 수행하지 않은 상태
- 203 : 요청을 성공하였지만, 요청에 대한 검증이 되지 않은 상태
- 204 : 요청을 성공했지만, 제공할 내용이 없음을 의미
- 205 : 204와 동일, 새로고침등을 통해 새로운 내용등을 확인할 것
- 206 : 요청의 일부분만 성공

### 300번대 : 요청을 마치기 위해 , 추가 조치가 필요

- 300 : 클라이언트가 동시에 여러 응답을 가르키는 URL을 요청한 경우 응답 목록과 함께 반환
- 301 : 요청한 URL이 옮겨졌을 때 사용, 옮겨진 URL에 대한 정보와 함께 응답
- 302 : 301과 동일, 하지만 여전히 옮겨지지 전 URL 요청
- 303 : 요청 받은 행동 수행을 위해서는 다른 URL로 요청
- 304 : 이전의 동일한 요청과 비교하여 변화가 없음을 의미(단시간에 반복된 동일 요청에 대한 대응 코드)
- 305 : 직접적인 요청이 아니라 반드시 프락시(우회경로)를 통해 요청되어야 됨

### 400번대 : 클라이언트 측 오류

- 400 : 잘못된 문법으로 인하여 서버가 요청을 이해할 수 없음
- 401 : 인증을 위해 권한 인증등을 요구
- 403 : 서버에 의해 요청이 거부
- **404 : 요청한 URL이 존재 X**
- 405 : 요청한 URL이 Method을 지원 X

### 500번대 : 서버 측 오류

- 500 : 서버에서 오류 발생
- 501 : 클라이언트 요청에 대한 서버의 응답 수행 기능이 없음
- 502 : 게이트웨이로 작업하는 동안 잘못된 응답을 수신했음을 의미
- 503 : 현재 서버가 요청 준비가 되지 않음, 유지보수나, 과부하로 인해 작동 중단되었을 때
- 504 : 서버가 게이트웨이 역할을 하고 있으며, 적시에 응답을 받을 수 없을 때
- 505 : 요청된 HTTP 버전은 서버에서 지원 X

### References

- [POST와 PUT 차이](https://blog.embian.com/66)
- [HTTP State Code](https://developer.mozilla.org/ko/docs/Web/HTTP/Status)