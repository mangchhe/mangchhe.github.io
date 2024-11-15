---
layout: post
title: Using Route 53 to Link Subdomains with Separate Name Servers
categories: devops
tags: devops
---

AWS에서 인프라를 구축하면서 개발 환경과 운영 환경을 서로 다른 계정으로 나누어 설정하였다. 대부분의 리소스는 동일하게 구성하되, 각 환경의 일부 리소스만 다르게 구성했다.

각 환경은 서로 다른 DNS를 사용하게 되었고, 운영 환경의 도메인은 `api.example.kr`, 개발 환경의 도메인은 `api.dev.example.kr`으로 Route 53에 등록하여 별다른 문제 없이 설정을 마친 것처럼 보였다.

그러나 운영 환경은 정상적으로 접근이 가능했지만, 개발 환경에서는 지속적으로 `NXDOMAIN` 오류가 발생하였다. 이 오류는 DNS에서 도메인을 찾을 수 없다는 의미였다.

문제의 원인을 파악해 본 결과, DNS 시스템이 **계층 구조**로 동작한다는 점을 이해하게 되었다. DNS는 상위 도메인에서 하위 도메인으로 요청을 전달하는 방식으로 작동한다. 그런데 나는 상위 도메인인 `example.kr`의 네임서버에 개발 환경의 하위 도메인인 `dev.example.kr`에 대한 네임서버(NS) 정보를 등록하지 않았던 것이다.

이로 인해, 상위 도메인 `example.kr`에서 하위 도메인 `dev.example.kr`로 요청이 전달되기 전에 차단되는 것이었다.

해결 방법은 간단했다. 운영 환경의 DNS에서 개발 환경의 네임서버(NS) 정보를 등록하여, 운영 계정의 네임서버가 개발 환경으로 요청을 전달할 수 있도록 설정했다. (**DNS 위임**(delegation))

이 작업을 완료한 후, `api.dev.example.kr` 도메인에 정상적으로 접근할 수 있었다.

**결론**

DNS는 계층적인 구조로 동작하며, 상위 도메인에서 하위 도메인의 네임서버로 요청을 위임해야만 하위 도메인이 올바르게 작동한다. 이 과정을 통해 개발 환경과 운영 환경이 각각 독립적인 DNS 설정을 가지고 있어도 문제없이 연결할 수 있었다.