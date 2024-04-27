---
layout: post
title: Sticky Session
categories: cs
tags: cs
---

세션 처리 시에 Scale out의 문제점 중 하나는 다중 서버 환경에서 사용자가 특정 서버에 세션을 가질 때 로드 밸런서에 의해 다른 서버로 라우팅 될 경우 세션이 끊어질 수 있다는 것이다. 이를 해결하기 위해 Sticky Session 방식을 사용할 수 있다.

Sticky Session은 로드 밸런서가 세션 기간 동안 동일한 클라이언트의 요청을 항상 동일한 서버로 라우팅해 주는 기능이다. 이를 통해 세션이 끊기지 않고 유지될 수 있다.

그러나 트래픽이 균등하게 분배되지 않아 특정 서버에 과부하가 발생할 수 있고 동일한 서버에게 보내기 위해 세션 정보에 대한 메타 데이터를 저장하고 있어야 하므로 더 많은 리소스를 필요로 한다. 그렇기 때문에 적절한 로드 밸런싱 전략을 도입하고 서버의 확장성을 고려하여 관리해야 한다.

> [https://medium.com/@mrcyna/what-are-the-sticky-sessions-222c378d2ce1](https://medium.com/@mrcyna/what-are-the-sticky-sessions-222c378d2ce1)