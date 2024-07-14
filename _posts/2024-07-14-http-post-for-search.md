---
layout: post
title: HTTP POST for Search
categories: web
tags: web
---

`GET` 메서드로 요청을 받을 때 요청 파라미터가 너무 복잡하고 길어지는 경우를 경험하게 되었다. 기본적으로 키-값 쌍의 형태를 가지는 `GET` 요청은 복잡한 데이터 구조나 대량의 데이터를 전달하기에는 적절하지 않다는 생각이 들었다.

`GET`은 기본적으로 캐싱을 할 수 있다고는 하지만 쿼리가 복잡해진 이상 캐싱을 수행할 수 있을까? 길어진 쿼리를 위해 URL 길이를 조정해야 할까? 로그를 봤을 때 파악하기 쉬운 형태의 URL이 될 수 있을까? 보안에 민감한 내용이 포함되어 있으면 어떻게 할까? 등 다양한 의문점이 생겼다.

그래서 `GET` 요청에 body를 담아서 보내는 것에 대해서 알아보았다.

> [RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.1)
> 
> **A payload within a GET request message has no defined semantics;**
> 
> **sending a payload body on a GET request might cause some existing implementations to reject the request.**
> 
> `GET`의 요청 메시지 payload는 정의된 의미가 없다. 
> 
> `GET` 요청으로 payload를 보내게 되면 일부 기존 구현체에서 요청을 거부할 수 있다.

`GET` 요청이 payload를 보내는 것이 불가능한 것은 아니나 제대로 본문을 처리할 수 없으니 권장하지 않는다는 것이다.

다른 방법으로 `POST` 메서드를 조회 쿼리로 사용하는 것은 괜찮아 보였다.

Restful 한 방법은 아닌 것 같지만 꽤 합리적인 방식인 것 같고 결국 restful의 설계 목적도 일관적인 컨벤션을 통해 프론트 개발자와의 높은 상호작용을 요구하는 것이니 사전에 이렇게 사용하는 것에 대해서 논의가 이루어지고 문서만 제대로 공유가 된다면 사용해도 된다고 생각되었다.

아래는 일부 회사들의 조회 API인데 `POST`로 조회 API를 사용하는 곳이 꽤 있는 것 같다. `POST`로 사용하는 정확한 이유는 궁금하지만, 알 방도가 없다..

- [https://developers.google.com/maps/documentation/places/web-service/nearby-search](https://developers.google.com/maps/documentation/places/web-service/nearby-search?hl=ko&_gl=1*1p9op73*_up*MQ..*_ga*ODM2OTU1NDI3LjE3MjA5NzE0Nzc.*_ga_NRWSTWS78N*MTcyMDk3MTQ3Ny4xLjAuMTcyMDk3MTQ5My4wLjAuMA..)
- [https://developers.kakaomobility.com/docs/navi-api/waypoints/](https://developers.kakaomobility.com/docs/navi-api/waypoints/)
- [https://tmapapi.tmapmobility.com/main.html#webservice/docs/tmapRouteSequential30](https://tmapapi.tmapmobility.com/main.html#webservice/docs/tmapRouteSequential30)