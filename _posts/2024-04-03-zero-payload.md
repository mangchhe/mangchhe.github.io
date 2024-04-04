---
layout: post
title: Zero-Payload
categories: cs
tags: cs
---

제로 페이로드 방식을 이벤트 기반 아키텍처에서 적용하면 다수의 이점을 얻을 수 있다.

이 방식에서는 이벤트 발행 시 최소한의 식별 정보만을 포함시키고 이를 수신한 후에 식별키를 이용해 관련 API를 호출하여 필요한 데이터를 얻게 된다.

이 방식은 이벤트의 순서를 유지하는 동시에 느슨한 결합을 가능하게 하여 효율적인 운용이 가능하다.

일반적으로 프로듀서는 파티션을 나누어 저장하고 컨슈머는 각 파티션의 데이터를 병렬로 읽기 때문에 순서를 보장하기 위해서는 별도의 처리가 필요하다. 하지만 제로 페이로드 방식을 이용하면 API를 통해 받은 응답 값으로 최신 데이터임을 보장할 수 있다.

이벤트를 여러 외부 시스템이 가져가서 사용하게 될 때 시스템마다 필요한 데이터를 포함해서 발행해야 하므로 비효율적이다. 목적에 맞게 API를 통해서 데이터를 전달받게 된다면 필요한 용도와 각 시스템의 비즈니스에 맞는 데이터와 기존 이벤트 데이터를 재설계할 필요가 없게 된다.

> [https://techblog.woowahan.com/7835/](https://techblog.woowahan.com/7835/)