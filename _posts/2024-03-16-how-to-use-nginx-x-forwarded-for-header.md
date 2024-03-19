---
layout: post
title: How to use nginx X-Forwarded-For Header?
categories: etc
tags: etc
---

**X-Forwarded-For (XFF)** 헤더는 프록시 서버를 통해 서버에 연결하는 클라이언트의 원래 IP 주소를 식별하기 위해 사용되는 일종의 표준 헤더이다.

클라이언트가 직접 서버에 연결하는 경우 클라이언트의 IP 주소는 서버로 전송된다. 그러나 클라이언트 연결이 프록시를 통과하는 경우에 서버는 마지막 프록시의 IP 주소만 알 수 있게 되어 유의미한 데이터가 아니게 된다. 

하지만 X-Forwarded-For 헤더를 이용하면 프록시나 로드 밸런서를 거칠 때마다 새로운 클라이언트 IP 주소를 덮어씌우는 게 아니라 목록에 누적시키면서 서버의 실제 IP 주소를 알 수 있게 된다.

```sh
X-Forwarded-For: client, proxy1, proxy2
```

따라서 서버에 필요한 클라이언트 IP 주소 정보를 제공하는 중요한 역할로 X-Forwarded-For 요청 헤더가 사용된다.

```sh
# Nginx에서 X-Forwarded-For 설정
# /etc/nginx/sites-available/default

server {
    ...
    location / {
        proxy_pass http://127.0.0.1:8080;
        ...
        # $proxy_add_x_forwarded_for 는 $remote_addr 값들을 이어 붙여 콤마로 구분한다. 
        # 만약 X-Forwarded-For 가 클라이언트 헤더에 존재하지 않으면 $proxy_add_x_forwarded_for, $remote_addr 둘은 같다.
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    ...
}
```

> [https://nginx.org/en/docs/http/ngx_http_proxy_module.html](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)