---
title: "[Spring Cloud] API Gateway Service란?"
description: API Gateway가 왜 필요한지, 무슨 역할을 하는지 알아보자
categories:
 - SpringCloud
tags:
 - Spring Cloud
 - MSA
 - API Gateway
---

# API Gateway Service란?

<hr>

사용자가 설정한 라우팅 설정에 따라서  End-Point로 클라이언트 대신에 요청하고 응답하면 다시 사용자에게 전달해주는 Proxy 역할을 해준다

# API Gateway가 왜 필요한가?

<hr>

![apiGateway_architecture](/assets/postImages/ApiGatewayConcept/apiGateway_architecture.png)

위 그림은 클라이언트 사이드에서 각 MicroSerivce 들을 호출하는 모습이다 서비스가 생성이 되거나 호스트네임이 변경이 되는 경우 서비스들은 독립적으로 빌드하고 배포할 수 있기 때문에 문제는 없다 하지만 문제는 클라이언트 사이드에서 발생한다

클라이언트 사이드에서 microservice의 주소들을 직접 이용해서 호출을 하였을 경우 클라이언트 앱과 브라우저도 같이 수정 배포가 이루어져야 했다 microservice가 추가, 변경이 이루어질 때마다 클라이언트 사이드가 동시에 같이 수정 - 빌드 - 배포가 이루어져야 하기 때문에 상호의존적이다

그러다보니 단일 진입점을 가지고 있는 형태로서 개발하는 것이 필요로 하게되었다

![apiGateway_architecture2](/assets/postImages/ApiGatewayConcept/apiGateway_architecture2.png)

서버단에서 단일 진입로를 가질 수 있는 API Gateway를 두고 각 microservice들에게 요청되는 모든 정보들에 대해서 클라이언트 사이드 종류에 상관없이 일괄적으로 처리할 수 있게 되었다

이렇게 클라이언트 사이드에 앱들은 각 microservice들을 상대할 필요 없이 API Gateway만 상대하면 되기 때문에 의사 전달이 더 수월해질 수 있다

# API Gateway의 특징

<hr>

- 인증 및 권한 부여
  - microservice마다 인증과 권한에 대해 구현하는 것이 아닌 단일 작업을 할 수 있다
- 서비스 검색 통합
  - microservice의 검색을 통합할 수가 있다
- 정책, 회로 차단기
  - 일괄적인 정책이나 문제가 발생한 microservice에 대해서 요청을 하지 않게 회로 차단기의 역할도 할 수 있다
- 부하 분산(Load Balancing)
- 로깅, 추적, 상관 관계
  - 각 microservice들의 로그들을 일괄적으로 관리할 수 있고 각 microservice 들은 서로 다른 microservice들을 연쇄적으로 호출하는 경우가 발생하는데 처음 진입점부터 시작해서 어디서 거치는지에 대한 추적을 할 수 있다
- IP 허용 목록에 추가
  - 일종의 방화벽 역할을 할 수가 있다

# Reference

<hr>

[이미지](https://docs.microsoft.com/ko-kr/dotnet/architecture/microservices/architect-microservice-container-applications/direct-client-to-microservice-communication-versus-the-api-gateway-pattern)
