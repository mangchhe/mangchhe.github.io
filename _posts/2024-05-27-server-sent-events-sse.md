---
layout: post
title: Server Sent Events (SSE)
categories: cs
tags: cs
---

**Server Push**

Server Push는 클라이언트가 아닌 서버가 통신을 시작하는 통신 방식을 말한다. 이 방식은 클라이언트가 통신을 시작하는 pull 방식과 다르다.

푸시 기술에서는 클라이언트가 특정 유형의 정보나 데이터를 받고자 하는 선호를 표현할 수 있으며 이는 publish–subscribe 모델을 통해 이루어진다. 이 모델에서는 클라이언트가 서버에서 제공하는 특정 채널을 구독하고 이 채널에 새로운 정보가 생기면 서버는 자동으로 해당 정보를 구독한 클라이언트에게 전송한다.

**Server-Sent Events**

<p align="center">
    <img src="/assets/postImages/ServerSentEventsSse/sse.png" width="650">
</p>

Server-Sent Events(SSE)는 서버 푸시 기술의 일종으로 HTTP 연결을 통해 클라이언트가 자동 업데이트를 받을 수 있게 한다. 이 기술은 클라이언트가 초기 연결을 설정한 후 서버가 클라이언트로 데이터를 전송할 수 있게 하고 주로 메시지 업데이트나 연속적인 데이터 스트림을 브라우저 클라이언트에 보내는 데 사용된다. 클라이언트는 EventSource API를 사용해 특정 URL을 요청하여 이벤트 스트림을 수신한다. 이때 SSE의 `Media-Type`은 `text/event-stream`이다.

> [https://en.wikipedia.org/wiki/Push_technology](https://en.wikipedia.org/wiki/Push_technology)
> 
> [https://en.wikipedia.org/wiki/Server-sent_events](https://en.wikipedia.org/wiki/Server-sent_events)
>
> [https://tecoble.techcourse.co.kr/post/2022-10-11-server-sent-events/](https://tecoble.techcourse.co.kr/post/2022-10-11-server-sent-events/)