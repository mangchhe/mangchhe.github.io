---
layout: post
title: Handling Split Logs with Fluent Bit on ECS
categories: devops
tags: devops, datadog, fluentbit, ecs
---

Datadog에서 하나의 로그가 **여러 개로 나뉘어 저장되는 현상**이 발생했다.
이로 인해 동일한 메시지가 조각나서 들어오며, 일부는 정상적으로 파싱되지만, 나머지는 `log` 필드에 **raw 문자열 전체가 담긴 형태**로 저장된다.

#### 로그 수집 구조

```
ECS Container → FireLens (Fluent Bit) → Datadog
```

#### 현상

Datadog에서 동일한 로그가 여러 줄로 저장되며, 일부는 정상적으로 파싱된 로그지만 나머지는 `log` 필드 하나에 전체 메시지가 문자열로 들어간 형태다.  

이는 Fluent Bit가 메시지를 **자동으로 분할하여 전송**하는 기능과 관련이 있다.

#### 원인

처음에는 Datadog을 의심했지만, 공식 문서에 따르면 로그가 256KB를 초과하는 경우 **자동으로 분할**된다고 명시되어 있다.
> [커스텀 로그 전달](https://docs.datadoghq.com/ko/logs/log_collection/?tab=http#%EC%BB%A4%EC%8A%A4%ED%85%80-%EB%A1%9C%EA%B7%B8-%EC%A0%84%EB%8B%AC)  
> 로깅을 위해 Datadog Agent를 사용하는 경우 **로그를 256kB(256000바이트)로 분할하도록 구성**됩니다.

실제 원인은 Datadog이 아니라 **Fluent Bit 쪽에서 먼저 로그를 분할하고 있었던 것**이다.

Fluent Bit는 메시지 크기가 **16KB를 초과하면 자동으로 분할**한다.  
이때 각 분할된 조각에는 다음과 같은 메타데이터가 포함된다:

```json
{
  "partial_message": true,
  "partial_id": "7bf32...",
  "partial_ordinal": 1,
  "partial_last": false
}
```

partial_message: 이 메시지가 분할된(partial) 메시지임을 나타냄
- partial_id: 동일한 원본 메시지를 식별하기 위한 ID
- partial_ordinal: 분할 순서를 나타냄
- partial_last: 이 조각이 마지막 메시지인지 여부

이러한 partial 메시지들이 Datadog으로 전송되면, Datadog은 이를 하나의 로그로 병합하지 않고 각각 개별 로그로 저장한다. 이로 인해 실제로는 하나의 메시지가 여러 개의 조각으로 나뉘어 저장되는 현상이 발생한다.

#### 해결 방법: Fluent Bit 설정으로 분할 로그 병합

이 문제는 Fluent Bit의 **multiline 필터**를 이용해 해결할 수 있다.
분할된 로그 메시지를 병합하는 방식은 AWS에서 제공하는 공식 예제에서도 확인할 수 있다:

> [FireLens Example: Concatenate Partial/Split Container Logs](https://github.com/aws-samples/amazon-ecs-firelens-examples/tree/mainline/examples/fluent-bit/filter-multiline-partial-message-mode)

##### 실제 적용한 설정 예시

```yml
[SERVICE]
    Flush              1
    Log_Level          debug
    Parsers_File       parsers.conf
    Buffer_Chunk_Size  64KB
    Buffer_Max_Size    256KB

[FILTER] # 추가
    Name               multiline
    Match              *
    Multiline.Key_Content log
    Mode               partial_message

[OUTPUT]
    Name               datadog
    Match              *
    Host               http-intake.logs.datadoghq.com
    TLS                On
    Apikey             {{api_key}}
    dd_service         {{service_name}}
    dd_source          ecs
    dd_tags            env:{{env}},team:{{team_name}}
```