---
title: "Apply to https to Nginx"
categories: web
tags: web
---

# HTTPS(HyperText Transfer Protocol Secure)란?

<hr>

HTTPS는 HTTP 통신 과정에 평문 데이터를 주고 받는 과정에 그대로 탈취&변조가 가능하다는 점과 같은 보안에 취약한 문제점들을 해결한 프로토콜이다.

![osi4plusssl](/assets/postImages/HttpsConcept/osi4plusssl.JPG)

네트워크 통신 과정을 나타낸 TCP/IP 4계층이며 SSL & TLS라는 보안 계층이 추가되어 암호화와 인증 등을 보장한다.

# 사전 지식

<hr>

동작 원리를 알아보기 전에 알아야 할 용어들을 알고 넘어가자

## 대칭키

대칭키는 암호화하거나 복호화할 때 사용되는 키가 같은 것을 의미한다. 대칭키의 단점으로 한쪽에서 키를 만들면 다른 쪽으로 안전하게 전달해주지 못하는 키 배송 문제가 있다.

## 공개키(=비대칭키)

공개키는 암호화할 때와 복호화할 때 사용되는 키가 다른 것을 의미한다. 서로 다른 두 키를 공개키와 개인키로 부르며 공개키로 암호화하면 개인키로만 복호화 가능하고 개인키로 암호화하면 공개키로만 복호화가 가능하다. 공개키의 단점으로 암호화 속도가 느리다.

## CA(Certificate authority)

클라이언트가 통신하고 있는 서버가 클라이언트가 의도한 서버가 맞는지 인증하기 위한 제 3자 기관이다.

# 동작 원리

<hr>

![sslhandshake](/assets/postImages/HttpsConcept/sslhandshake.JPG)

**① [Client Hello]** 클라이언트는 브라우저가 사용하는 SSL & TLS 버전 정보, 브라우저가 지원하는 암호화 방식, 브라우저가 생성한 임의의 난수를 서버에 전송한다.

**② [Server Hello]** 서버는 클라이언트가 보내온 암호화 방식 중에서 하나를 선택하고 CA로 부터 받은 SSL 인증서(서버 공개키가 암호화 되어있음), 서버가 생성한 임의의 난수를 클라이언트에 전송한다.

**③ [서버 인증서 확인]** 클라이언트는 브라우저에 내장된 CA 리스트, CA 공개키를 가지고 서버에게 받은 SSL 인증서와 같은 CA를 찾아 공개키로 복호화한다. SSL 인증서를 제대로 복호화 하였다면 신뢰할 수 있는 인증서임을 증명한다. (SSL 인증서는 CA 비밀키로 암호화가 되어있기 때문에 해당 CA 공개키로만 복호화가 가능하다.)

**④ [대칭키 전달]** 클라이언트는 아까 생성한 난수와 서버에서 받은 난수를 가지고 대칭키를 생성하고 복호화된 공개키를 가지고 대칭키를 암호화하여 서버에게 전송한다.

**⑤ [공개키 복호화]** 서버는 가지고 있는 비밀키를 가지고 공개키로 암호화된 클라이언트 대칭키를 복호화한다.

**⑥ [HTTPS 통신]** 클라이언트, 서버는 같은 대칭키로 암/복호화하며 통신을 진행할 수 있다.

# Wireshark로 보는 http, https 보안 차이점

<hr>

wireshark로 http와 https일 때 각각 POST 요청을 보내어 어떻게 다른지 확인해보자

``` java
@RestController
public class TestController {

    @PostMapping("/signup")
    public String signup(@RequestBody TestDto testDto) {
        System.out.println(testDto.id + " " + testDto.pw);
        return "success";
    }

}
@Getter @Setter
public class TestDto {

    String id;
    String pw;
}
```

간단하게 Spring Boot로 아이디와 비밀번호를 받아 회원가입을 하는 controller와 dto 생성한 다음 서버로 가서 어플리케이션을 실행시켜준다. 물론 실제 회원가입 로직을 짜진 않았다.

![postmansighup](/assets/postImages/HttpsConcept/postmansighup.JPG)

postman을 이용해서 http, https 각각 요청을 보낼 준비를 한다.

![wiresharkHttp](/assets/postImages/HttpsConcept/wiresharkHttp.JPG)

첫 번째로 http로 Post 요청을 보냈다. 아마 http 통신 과정을 한번 공부해봤다면 익숙한 용어들이 보이게 될 것이다. 모르겠다면 -> [HTTP 동작 과정](https://mangchhe.github.io/web/2021/02/19/HttpActionProcess/)

3-way handshake를 거쳐 연결을 수립하고 요청과 응답이 오가고 keep-alive가 유지가 되는 것 같다. 사실 keep-alive가 어떻게 유지되는지까지는 알지 못한다.

![wiresharkHttp2](/assets/postImages/HttpsConcept/wiresharkHttp2.JPG)

요청 Body를 확인해보면 내가 postman을 통해 body에 담아 요청 보냈던 내용들이 내용들이 엄청 허무하게 보이는 것을 확인할 수 있다. 이것이 실제 개인의 개인정보라면 엄청난 보안 이슈가 될 것이다.

자 그러면 https를 살펴보자

![wiresharkHttps](/assets/postImages/HttpsConcept/wiresharkHttps.JPG)

위에서 살펴봤던대로 client hello, server hello가 실행되고 그 다음 줄부터 Change Cipher Spec, 위에서 봤던 공개키로 암호화하여 대칭키를 전달하는 과정인 것 같다. 이 부분을 보면 Application Data도 함께 들어있는데 대칭키로 암호화하여 요청 데이터를 같이 보내는 것으로 확인할 수 있다. 그다음 Apllication Data는 대칭키로 암호화한 클라이언트에게 보내는 응답 메세지가 될 것이다.

![wiresharkHttps2](/assets/postImages/HttpsConcept/wiresharkHttps2.JPG)

Application Data를 http와 같이 읽어보기 위해서 확인해본 결과 http와는 다르게 내용들이 암호화되어 있어서 확인이 불가능한 것으로 확인할 수 있다.

이정도면... https를 적용해야되는 이유가 설명이 된다..??

# Nginx 적용

<hr>

해당 내용은 Ubuntu 18.04 LTS에서 진행된다.

인증 기관(CA)에 인증서를 발급받기 위해서 Let's Encrypt 라는 곳에서 무료로 인증서를 발급받자

``` bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt update
sudo apt install certbot -y
```

apt repository에 cerbot을 생성한 후에 cerbot을 설치한다.

``` bash
sudo certbot certonly --standalone -d <도메인명>
```

위 명령어를 작성하게 되면 /etc/letsencrypt/live/<도메인명>/ 폴더 내에 privkey.pem, fullchain.pem 파일들이 생성된 것을 볼 수 있다.


``` nginx
{
        listen 80;
        server_name <도메인명>;
        return 301 https://$server_name$request_uri;
}

server {
        listen 443 http2;
        server_name <도메인명>;

        ssl on;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_certificate /etc/letsencrypt/live/<도메인명>/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/<도메인명>/privkey.pem;

        location / {
                proxy_pass http://localhost:8080;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }
}
```

/etc/nginx/site-enabled/ 폴더에 들어가 설정 파일을 위와 같이 수정해주면 https로 접속이 되는 것을 확인할 수 있다.

설정 하는 방법을 모르겠다면 -> [링크](https://mangchhe.github.io/was/2021/07/28/NginxConcept/)

![sslresult](/assets/postImages/HttpsConcept/sslresult.JPG)

그림과 같이 HTTPS가 적용된 것을 확인할 수 있고 인증서에 들어가보면 발급자가 Let's Encrypt로 되어있는 것을 확인할 수 있다.
