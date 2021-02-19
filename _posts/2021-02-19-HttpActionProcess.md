---
title: 웹 단계별 동작 과정
decription: 웹에서 브라우저 시작으로 요청 메시지를 보내고 서버에서 응답 메시지를 보내기까지의 HTTP 통신에 필요한 동작 과정들에 한 단계씩 알아보자
categories:
 - WEB
tags:
 - WEB
 - HTTP
---

> 웹에서 브라우저 시작으로 요청 메시지를 보내고 서버에서 응답 메시지를 보내기까지의 HTTP 통신에 필요한 동작 과정들에 한 단계씩 알아보자

> ## 웹 동작 과정

클라이언트 -> 요청 -> 서버 -> 응답 -> 클라이언트 과정에 필요한 동작

> ### 약식 설명

![httpProcess](/assets/httpProcess.jpg)

> ### 1. 검색창에 URL 주소 입력

사람들은 자기가 원하는 사이트로 가기 위해서 웹브라우저를 열어 주소창에 해당 주소를 입력한다

> #### URL 주소 형식

![url](/assets/url.PNG)

> ### 2. DNS 서버에서 도메인 네임 검색

물리적 주소인 IP는 mangchhe.github.io와 같이 사람이 알아볼 수 있는 문자로 변환되어 DNS 서버에서 등록되어 있다

네이버를 예로 들어보자

네이버를 검색할 때 www.naver.com 라고 적을 것이다 이 도메인을 보면 누가봐도 네이버 주소라는 것을 알 수 있을 것이다

![navernslookup](/assets/navernslookup.PNG)

원래 네이버에 접속하기 위해서는 위 사진에 있는 IP 주소를 입력하여 매번 접속을 시도해야되지만 해당 주소에 대해서 DNS 서버에 도메인 네임으로 naver.com을 등록해두었기 때문에 IP를 알지 못하여도 접속이 가능하다

본론으로 돌아와 **DNS 서버에서 사용자가 적은 URL 주소에 도메인 네임 부분에 대한 IP 주소를 찾아낸다**

> ### 3. 클라이언트 - 서버 TCP 연결 시도

HTTP 메시지를 전송하기 전에 클라이언트와 서버 간에 안전한 통신 터널을 만든다

**3-way handshake** : 클라이언트 - 서버간에 신뢰성 있는 연결을 위해 3번의 패킷 교환 과정

1. **Client -> Server** : 접속 요청하는 SYN 패킷 전송
2. **Server -> Client** : 접속 요청을 수락했고, 접속 요청 하는 쪽에도 포트를 열어달라는 SYN, ACK 패킷 전송
3. **Client -> Server** : 연락 정상적으로 이루어졌다는 ACK 패킷 전송

**패킷** : 데이터를 묶음 단위로 한 번에 전송할 데이터의 크기를 의미, 웹 통신은 3계층인 네트워크 계층에서 이루어지며 묶음을 패킷이라고 부른다

> ### 4. HTTP 메시지 요청

Header, Body 구조로 이루어진 HTTP 메시지를 요청한다

![httpMessageRequest](/assets/httpMessageRequest.PNG)

> #### Header

- Host : 요청이 전송되는 target 호스트 URL 주소
- User-Agent : 요청을 보내는 웹 브라우저의 정보
- Aceept : 응답 Body 데이터 타입 정보
- Connetion : 해당 요청이 끝난 후에 계속 연결을 유지 할 것인지 끊을 것인지 정보
  - keep-alive : 한번 연결 후 일정 시간안에 계속 요청이 있을 시 끊지 않고 유지함
  - close : 한번 연결 후 바로 끊음
- Content-Type : HTTP 요청 보내는 메시지 Body 타입
- Content-Length : 요청 보내는 메시지 Body의 총 사이즈

> #### Body

- HTTP 요청 또는 응답에 전송할 데이터를 담는 부분
- 전송할 데이터가 존재하지 않는다면 비어있다

> ### 5. 클라이언트 - 서버 TCP 연결 해제

이전에 만든 통신 터널을 해제한다

**4-way handshake** : TCP 연결을 해제하기 위해 수행되는 절차

1. **Client -> Server** : 연결을 종료하겠다는 FIN 패킷 전송
2. **Server -> Client** : 연결 종료를 확인했다는 ACK 전송, 자신의 통신이 끝날때까지 기다림
3. **Server -> Client** : 통신이 끝났을 경우 FIN 패킷 전송
4. **Client -> Server** : 통신 해지를 확인했다는 ACK 패킷 전송

> ### 6. 비즈니스 로직 동작

요청 메시지에 따라 서버에서 응답에 필요한 데이터를 가공하는 작업이 필요로함

> ### 7. 클라이언트 - 서버 TCP 연결 시도

앞서 설명과 동일

> ### 8. HTTP 메시지 응답

서버에서 HTTP 메시지를 받게 되면 어떤 메소드 요청인지, 어떤 리소스를 찾는지 확인한 후 찾아서 해당 리소스와 응답코드를 함께 클라이언트에게 전달하게 된다

> #### Status Line

- HTTP Version : HTTP 버젼
- Status code : 응답 상태 코드
- Statue Text : 응답 상태 부가 설명

![httpMessageResponse](/assets/httpMessageResponse.PNG)

> #### Header

앞서 설명과 동일하나, User-Agent 대신 Server헤더가 사용

> #### Body

앞서 설명과 동일

> ### 9. 클라이언트 - 서버 TCP 연결 해제

앞서 설명과 동일

> ### 10. 웹 브라우저에 웹 문서 출력

웹 브라우저 화면에 응답 받은 메시지의 Body 부분에 있는 데이터를 나타냄
