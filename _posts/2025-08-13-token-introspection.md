---
layout: post
title: OAuth 2.0 Token Introspection
categories: cs
tags: cs
---

OAuth 2.0에서 토큰의 유효성과 메타데이터를 검증하는 표준 메커니즘이다. Resource Server가 Authorization Server에게 토큰 상태를 질의하여 active 여부, scope, 만료 시간 등의 정보를 획득한다.

```http
POST /introspect HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

token=mF_9.B5f-4.1JqM&token_type_hint=access_token
```

이렇게 요청하면 Authorization Server는 토큰 정보를 JSON으로 돌려준다. `active: true`면 유효한 토큰이고, `scope`, `exp`, `iat` 등의 메타데이터도 함께 온다.

JWT는 토큰 자체에 정보가 들어있지만(self-contained), opaque 토큰은 그냥 랜덤 문자열이라 서버에 물어봐야 한다. Token Introspection은 이런 opaque 토큰을 검증할 때 쓰는 표준 방법이다.

#### JWT vs Opaque 토큰 비교

| 특징 | JWT | Opaque 토큰 |
|------|-----|-------------|
| **구조** | JSON 형태, 자체 포함형 | 불투명한 랜덤 문자열 |
| **정보 저장** | 토큰 내부에 클레임 포함 | 서버 측에 별도 저장 |
| **검증 방식** | 서명 검증으로 자체 검증 | Authorization Server 질의 필요 |
| **크기** | 클레임에 따라 상대적으로 큼 | 작고 고정된 크기 |
| **보안성** | 기본적으로 인코딩만 (암호화 가능) | 기본적으로 불투명 |
| **토큰 철회** | 불가능 (만료까지 유효) | 즉시 철회 가능 |

#### Opaque 토큰을 써야 하는 경우

**민감한 정보 전송할 때**
- JWT는 기본적으로 인코딩만 되어 있어서 누구나 읽을 수 있음
- 민감한 데이터는 opaque 토큰으로 숨기는 게 안전

**토큰 철회가 필요할 때**
- JWT는 한번 발급되면 만료까지 못 바꿈
- Opaque 토큰은 서버에서 바로 무효화 가능
- 세션 관리나 긴급 권한 해제에 유용

**페이로드 크기 때문에**
- 큰 JWT는 대역폭 많이 먹음
- Opaque 토큰은 작은 문자열만 주고받으면 됨

#### JWT를 써야 하는 경우

**분산 시스템에서**
- 매번 Authorization Server 질의하면 병목 생김
- JWT는 각 서비스에서 독립적으로 검증 가능

**단순한 용도로**
- 민감하지 않은 정보
- 토큰 철회 필요 없음
- 빠른 검증이 중요

> Opaque 토큰: 토큰 자체로는 의미를 알 수 없는 불투명한 문자열. JWT와 달리 토큰 내부에 정보가 없어서 Authorization Server에 질의해야 토큰 상태를 알 수 있다.

endpoint는 TLS 필수이고, 토큰 스캐닝 공격 방지를 위해 인증이 필요하다. 비활성 토큰에는 `active: false`만 반환한다.

> 토큰 스캐닝 공격: 공격자가 무작위 토큰 값들을 시도해서 유효한 토큰을 찾아내는 공격. Introspection endpoint에 인증 없이 접근할 수 있다면 이런 공격이 가능하다.

> - [OAuth 2.0 Token Introspection - RFC7662](https://datatracker.ietf.org/doc/html/rfc7662) 
> - [JWT vs Opaque Tokens](https://zitadel.com/blog/jwt-vs-opaque-tokens)
> - [JWT vs Opaque Tokens - Martin Hong](https://martinhongsw.medium.com/jwt-vs-opaque-tokens-%EB%B9%84%EA%B5%90-8fa5ff6da2f8)