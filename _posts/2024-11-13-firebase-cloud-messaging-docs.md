---
layout: post
title: Firebase Cloud Messaging (FCM) Docs
categories: etc
tags: etc
---

> [메시지 유형](https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ko&_gl=1*krsf23*_up*MQ..*_ga*MjEyODExMDY1NC4xNzMwMTc3NjAy*_ga_CW55HF8NVT*MTczMDE3NzYwMS4xLjAuMTczMDE3NzYwMS4wLjAuMA..#notifications_and_data_messages)
>

- 알림 메시지: 종종 '표시 메시지'로 간주. FCM SDK에서 자동으로 처리
- 데이터 메시지 : 클라이언트 앱에서 처리
- 백그라운드에서 FCM SDK가 알림 표시를 자동으로 처리하게 하려면 알림 메시지 사용, 자체 클라이언트 앱 코드로 메시지를 처리하려면 데이터 메시지 사용
- 혼합해서 보낼 수 있고 알림 페이로드 표시는 FCM, 데이터 페이로드는 클라이언트 앱에서 처리
  - 알림과 데이터 페이로드가 모두 포함된 메시지를 수신한 경우의 앱 동작은 앱이 백그라운드 상태인지 아니면 포그라운드 상태인지에 따라 다르다.
  - 백그라운드 상태인 경우 알림 페이로드가 앱의 알림 목록에 수신되며 사용자가 알림을 탭한 경우에만 앱에서 데이터 페이로드를 처리
  - 포그라운드 상태인 경우 앱에서 페이로드가 둘 다 제공되는 메시지 객체를 수신

```yml
{
  "message":{
    "token":"bk3RNwTe3H0:CI2k_HHwgIpoDKCIZvvDMExUdFQ3P1...",
    "notification":{
      "title":"Portugal vs. Denmark",
      "body":"great match!"
    },
    "data" : {
      "Nick" : "Mario",
      "Room" : "PortugalVSDenmark"
    }
  }
}
```

> [여러 플랫폼의 메시지 맞춤 설정](https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ko&_gl=1*krsf23*_up*MQ..*_ga*MjEyODExMDY1NC4xNzMwMTc3NjAy*_ga_CW55HF8NVT*MTczMDE3NzYwMS4xLjAuMTczMDE3NzYwMS4wLjAuMA..#customizing-a-message-across-platforms)

- 플랫폼별 전송 옵션이 존재

```yml
{
  "message":{
     "token":"bk3RNwTe3H0:CI2k_HHwgIpoDKCIZvvDMExUdFQ3P1...",
     "notification":{
       "title":"Match update",
       "body":"Arsenal goal in added time, score is now 3-0"
     },
     "android":{
       "ttl":"86400s",
       "notification"{
         "click_action":"OPEN_ACTIVITY_1"
       }
     },
     "apns": {
       "headers": {
         "apns-priority": "5",
       },
       "payload": {
         "aps": {
           "category": "NEW_MESSAGE_CATEGORY"
         }
       }
     },
     "webpush":{
       "headers":{
         "TTL":"86400"
       }
     }
   }
 }
```

> [비축소형 메시지, 축소형 메시지](https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ko&_gl=1*krsf23*_up*MQ..*_ga*MjEyODExMDY1NC4xNzMwMTc3NjAy*_ga_CW55HF8NVT*MTczMDE3NzYwMS4xLjAuMTczMDE3NzYwMS4wLjAuMA..#delivery-options)

- 비축소형 메시지
  - 유용한 콘텐츠를 전송
  - 대표적인 사용 사례는 채팅 메시지나 중요한 메시지
- 축소형 메시지
  - 모바일 앱으로 콘텐츠 없이 '핑'을 보내는 것
  - 일반적인 사례로 모바일 앱에 서버의 데이터와 동기화할 것을 알리는 데 사용하는 메시지

> [FCM 등록 토큰 관리를 위한 권장 사항](https://firebase.google.com/docs/cloud-messaging/manage-tokens?hl=ko)

- FCM에 1개월 넘게 연결되지 않으면 비활성 토큰으로 간주
  - 이러한 토큰은 메시지 전송 및 주제 팬아웃이 안될 수 있다.

> **topic fanouts (팬 아웃)**
>
> 주제에 메시지가 게시될 때, 해당 주제를 구독한 모든 활성 토큰으로 메시지 전송하는 것을 말한다.

- 토큰이 270일 동안 사용되지 않아 비활성 상태에 도달하면 만료된 토큰으로 간주
- 서버에서 중요한 역할은 토큰을 추적하고 활성 토큰의 업데이트된 목록을 유지하는 것이다.
  - 토큰 타임스탬프를 따로 구현하고 정기적으로 업데이트하는 것이 좋다.

> [토큰 최신 상태 유지 및 비활성 토큰 삭제](https://firebase.google.com/docs/cloud-messaging/manage-tokens?hl=en#ensuring-registration-token-freshness)

- 서버에서 토큰 유효성을 확인하는 방법은 메시지를 보냈을 때 오류 메시지를 통해 알 수 있다.
  - `UNREGISTERED (404)`
  - `INVALID_ARGUMENT (400)`
- 타임스탬프를 관리하고 있다면 270일이 지났다는 것을 판단하고 비활성 토큰을 삭제할 수 있다.

> [메시지의 수명](https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ko&_gl=1*x8par1*_up*MQ..*_ga*NzIwODMwMTM5LjE3MzAzNjA2MTA.*_ga_CW55HF8NVT*MTczMDM2MDYxMC4xLjAuMTczMDM2MDYxMC4wLjAuMA..#lifetime)

- FCM에 메시지를 게시한 후 메시지 ID를 수신했을 때 메시지가 기기로 전송되었다는 의미는 아니다. 정확히는 전송이 수락되었다는 의미이다.

- 메시지 정보 확인 세 가지 방법
  - Firebase Console 메시지 전달 보고서 패널 확인
  - FCM Data API
  - BigQuery
- BigQuery만 개별 메시지 이벤트 정보를 확인할 수 있고 나머지는 통계 데이터만 확인할 수 있다.

> [BigQuery 내보내기 데이터 설명](https://firebase.google.com/docs/cloud-messaging/understand-delivery?hl=ko&platform=ios#how_does_this_data_differ_from_data_exported_to_bigquery)
>
> [BigQuery 데이터 스키마](https://firebase.google.com/docs/cloud-messaging/understand-delivery?hl=ko&platform=ios#what-data-exported)