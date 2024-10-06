---
layout: post
title: DNS Record
categories: cs
tags: cs
---

인터넷에서 사용자가 도메인을 통해 웹페이즈를 접속하거나 특정 서비스와 통신할 때, 네임 서버에 질의를 하여 설정된 값을 찾아 통신하게 된다.

이때, 네임 서버에 설정된 항목들을 레코드라고 한다.

> [DNS (Domain Name System)](https://www.ibm.com/kr-ko/topics/dns)
>
> 사용자에게 친숙한 도메인 이름을 컴퓨터가 네트워크에서 서로를 식별하는 데 사용하는 인터넷 프로토콜(IP) 주소로 변환하는 인터넷 표준 프로토콜의 구성 요소이다.

**A 레코드**

도메인 이름을 IPv4 주소와 매핑하는 레코드이다. 도메인과 아이피 간의 매칭은 1:N 일 수 있다.

```sh
$ nslookup naver.com
Server:		210.220.163.82
Address:	210.220.163.82#53

Non-authoritative answer:
Name:	naver.com
Address: 223.130.192.248
Name:	naver.com
Address: 223.130.200.236
Name:	naver.com
Address: 223.130.192.247
Name:	naver.com
Address: 223.130.200.219
```

**AAAA 레코드**

A 레코드와 비슷하지만, IPv6 주소를 사용한다.

**CNAME 레코드**

별칭 도메인 이름을 다른 도메인 이름에 매핑한다. 예를 들어 joohyun.com → joohyun2.com 과 같이 다른 도메인 주소로 연결하는 방식이다.

**MX(Mail Exchanger) 레코드**

메일 서버를 지정하는 레코드이며, 도메인으로 수신되는 메일을 처리할 서버를 나타낸다.

**NS (Name Server) 레코드**

도메인에 대한 권한이 있는 네임 서버를 지정하는 레코드이다.

**SOA (Start of Authority) 레코드**

네임 서버가 해당 도메인에 관하여 인증된 데이터를 가지고 있음을 증명하는 레코드이다.

**TXT (Text) 레코드**

도메인에 대한 임의의 텍스트 정보를 저장할 수 있는 레코드이다. 인증이나 검증 목적으로 사용된다.


> - [https://www.whatsmydns.net/](https://www.whatsmydns.net/)
> - [https://toolbox.googleapps.com/apps/dig/](https://toolbox.googleapps.com/apps/dig/)