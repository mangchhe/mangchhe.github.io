---
layout: post
title: API Rate Limiting
categories: cs
tags: cs
---

API 요청 수를 제한해서 서버 리소스를 보호하고 공정한 사용을 보장하는 메커니즘이다. 무분별한 요청으로 인한 서버 과부하를 방지하고, 악의적인 공격이나 남용을 막는 핵심 기술이다.

#### 필요한 이유

**서버 리소스 보호**
- CPU, 메모리, 데이터베이스 연결 등 한정된 리소스 관리
- 갑작스런 트래픽 스파이크로부터 시스템 안정성 확보

**공정한 사용 보장**
- 한 사용자가 모든 리소스를 독점하는 것 방지
- 여러 클라이언트 간 공평한 서비스 제공

**비용 관리**
- 클라우드 환경에서 과도한 리소스 사용으로 인한 비용 폭증 방지
- 외부 API 호출 비용 제어

#### Rate Limiting 알고리즘

**Token Bucket**

가장 널리 쓰이는 방식. 토큰이 담긴 버킷에서 요청마다 토큰을 소모한다.

```
버킷 크기: 10개
토큰 보충 속도: 1개/초

요청 5개 → 토큰 5개 소모 → 남은 토큰: 5개
1초 후 → 토큰 1개 보충 → 남은 토큰: 6개
```

- **장점**: 버스트 트래픽 허용, 구현 간단
- **단점**: 메모리 사용량 증가

**Leaky Bucket**

물이 새는 양동이처럼 일정한 속도로만 요청을 처리한다.

```
처리 속도: 1개/초 고정
대기열 크기: 5개

요청 10개 들어와도 → 1초에 1개씩만 처리
나머지는 대기열에서 순서 대기
```

- **장점**: 일정한 처리 속도 보장
- **단점**: 버스트 트래픽 대응 어려움

**Fixed Window**

고정된 시간 구간에서 요청 수를 카운트한다.

```
1분 구간에서 100개 제한
00:00~00:59 → 100개 허용
01:00~01:59 → 다시 100개 허용
```

- **장점**: 구현 매우 간단
- **단점**: 경계 시점에서 2배 트래픽 가능

**Sliding Window**

시간 구간이 요청 시점에 따라 움직인다.

```
1분 구간에서 100개 제한
12:30:15 요청 시 → 12:29:15~12:30:15 구간 확인
12:30:45 요청 시 → 12:29:45~12:30:45 구간 확인
```

- **장점**: 정확한 제한, 경계 문제 해결
- **단점**: 구현 복잡, 메모리 사용량 많음

#### HTTP 응답 처리

**429 Too Many Requests**

Rate limit 초과 시 표준 응답이다.

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640995200

{
  "error": "Rate limit exceeded",
  "message": "Try again in 60 seconds"
}
```

**주요 헤더들**

- `X-RateLimit-*` 헤더들은 사실상 표준이지만 공식 RFC는 아님
- 각 서비스마다 미묘하게 다른 구현으로 상호 운용성 문제 존재
- 기존에 널리 쓰이는 헤더들 (비표준)
  - X-RateLimit-Limit: 시간당 허용 요청 수
  - X-RateLimit-Remaining: 남은 요청 수  
  - X-RateLimit-Reset: 제한 리셋 시점
  - Retry-After: 다음 요청까지 대기 시간 (RFC 9110 표준)

#### 실제 서비스 사례

**[GitHub API](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api)**
```http
x-ratelimit-limit: 5000
x-ratelimit-remaining: 4999
x-ratelimit-reset: 1640995200
x-ratelimit-used: 1
x-ratelimit-resource: core
```

**[Twitter API](https://developer.x.com/en/docs/x-api/rate-limits)**
```http
x-rate-limit-limit: 300
x-rate-limit-remaining: 299
x-rate-limit-reset: 1640995200
```

각 서비스마다 미묘하게 다른 헤더 이름과 값 형식을 사용하고 있어서, 표준화가 필요한 상황이다.

> - [RFC 6585 - Additional HTTP Status Codes](https://tools.ietf.org/html/rfc6585)
> - [HTTP 429 Too Many Requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429)
> - [draft-ietf-httpapi-ratelimit-headers](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/)
