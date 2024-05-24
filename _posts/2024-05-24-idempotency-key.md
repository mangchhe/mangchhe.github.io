---
layout: post
title: Idempotency Key
categories: cs
tags: cs
---

**멱등성(Idempotency)**

시스템에서는 함수 호출이 성공하거나 실패할 수 있다. 하지만 컴포넌트(서버)로 호출할 시에 새로운 상황이 있을 수 있다.

- 호출이 원격 컴포넌트에 도달하지 못한 경우
- 컴포넌트에서 처리 중 문제가 발생한 경우
- 원격 컴포넌트의 응답이 클라이언트에 도달하지 못한 경우

이 상황에서 유일한 방법은 요청을 다시 보내는 것이다. 하지만, 단순히 조회하는 것은 괜찮을지도 몰라도 업데이트하는 요청은 고려해야 할 부분들이 생기고 복잡해질 것이다.

여기서 멱등성이라는 개념이 필요하다.

> [Idempotence](https://en.wikipedia.org/wiki/Idempotence)
>
> 초기 적용 이후 결과를 변경하지 않고 여러 번 적용할 수 있는 수학과 컴퓨터 과학의 특정 연산 속성

HTTP API 영역에서의 멱등성 여부

- GET, PUT, DELETE, HEAD, OPTIONS, OPTIONS : 멱등성 O
- POST, PATCH : 멱등성 X

**솔루션**

클라이언트가 요청할 때 고유한 키를 함께 보내고 서버는 키와 요청 쌍으로 트래킹한다.

- 서버에 이미 해당 값이 있으면 요청을 무시한다.
- 서버에 해당 값에 대한 기록이 없으면 값을 저장한다.

위 아이디어는 IETF `The Idempotency-Key HTTP Header Field` 스펙이다.

다이어그램에서 클라이언트가 유니크 키를 생성하고 서버에 보내게 되면 추출하여 값이 존재하면 캐싱해 둔 응답을 내려주고 그렇지 않으면 새로 생성하여 응답을 내려준다.

<p align="center">
    <img src="/assets/postImages/IdempotencyKey/idempotency.png" width="450">
</p>

해당 스펙에서 발생할 수 있는 오류 시나리오는 세 가지이다.

- 400 : Idempotency key가 누락된 경우
- 422 : 다른 요청에 Idempotency key를 사용하려고 한 경우
- 409 : 기존 요청이 여전히 처리 중인데 재시도한 경우

결론은 문제 상황에서 방법은 재호출 방식밖에 없지만 멱등성이 아닌 작업에서 두 번 실행하면 문제가 생길 수 있다.

REST API에서 Idempotency-Key 헤더에 넣어 위험을 방지할 수 있다.

> [https://medium.com/apache-apisix/fixing-duplicate-api-requests-05a84caeceac](https://medium.com/apache-apisix/fixing-duplicate-api-requests-05a84caeceac)