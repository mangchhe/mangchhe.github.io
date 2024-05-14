---
layout: post
title: Various Ways to Create an ID
categories: cs
tags: cs
---

**Oracle Sequence 및 MySQL Auto Increment**

Oracle의 Sequence와 MySQL의 Auto Increment는 데이터베이스가 자동으로 순차적인 ID 값을 생성하는 기능이다. 이 방법은 간단하며 초기 설계에서 쉽게 구현할 수 있다. 하지만, 매우 높은 트래픽 상황에서는 ID 생성에 부하가 걸릴 수 있으며 Oracle RAC와 같은 분산 환경에서는 항상 완벽히 순차적이지 않을 수 있다. ID는 기본적으로 8바이트 또는 4바이트 길이를 사용한다.

> [Oracle RAC(Real Application Clusters)](https://www.oracle.com/kr/database/real-application-clusters/)
>
> 여러 서버에서 단일 Oracle Database를 실행함으로써 공유 스토리지에 액세스하는 동안 가용성을 극대화하고 수평 확장성을 구현할 수 있다. 각 서버가 독립적인 시퀀스를 사용하여 ID를 생성함으로써 네트워크 지연이나 처리 순서에 따라 순차적으로 ID가 생성되지 않을 수 있다.

**UUID (Universally Unique Identifier)**

UUID는 서버에서 생성되며 데이터베이스 부하 없이 임의의 값을 생성한다. 자바에서 기본적으로 사용하는 v4 UUID는 32개의 16진수와 4개의 대시로 이루어진 36글자의 문자열(128비트, 16바이트)로 중복될 가능성이 없다. 이는 분산 환경에서 적합하다.

클러스터형 인덱스

클러스터형 인덱스는 키 값을 기준으로 데이터를 정렬하여 저장한다. 주로 기본키가 이러한 인덱스 유형을 사용한다. 랜덤하게 생성되는 UUID는 데이터의 삽입 및 삭제 과정에서 인덱스 및 데이터의 빈번한 이동을 유발할 수 있으며 이는 특히 최신 데이터를 자주 조회하는 시나리오에서 데이터 캐시 효율을 저하시킬 수 있다.

**UUID v7**

UUID v7은 클러스터형 인덱스의 단점을 보완한다. 앞자리 48비트에 타임스탬프를 포함하여 시간 기준으로 정렬된 순서로 ID를 생성한다. 이는 클러스터형 인덱스에 적합하며, 여전히 36글자의 길이를 유지합니다.

**Snowflake**

![snowflake](/assets/postImages/VariousWaysToCreateAnId/snowflake.png)

- 부호비트(1) : 예약 비트(항상 0)
- 타임스탬프(41) : 밀리초 단위의 epoch timestamp. 시작 시점부터 대략 70년 사용 가능
- 머신ID(10) : 1024개의 개별 프로세스 수용
- 시퀀스(12) : 각 프로세스 당 1씩 증가하여 카운트. 밀리초마다 0으로 재설정

Snowflake ID는 64비트 (8바이트) 구조로, 구 트위터에서 개발되었다. 이 구조는 타임스탬프, 머신 ID, 시퀀스를 포함하여 한 머신 ID 기준으로 초당 최대 40만개의 ID를 생성할 수 있다. 이 ID 생성 방식은 약 70년 동안 사용할 수 있으며, 대규모 시스템에 적합하다.

**결론**

글로벌 서비스가 아니며 트래픽이 많지 않을 경우 간단한 시퀀스나 자동 증가 방식이 충분할 수 있다. `YYYYMMDDHHMM-랜덤` 형식의 ID도 유효한 대안이 될 수 있고 최신 데이터를 빈번하게 읽는 시나리오나 데이터 쓰기가 많은 경우 정렬된 순서로 생성되는 ID (예: 클러스터형 인덱스)를 고려하는 것이 좋다.

> [https://www.youtube.com/watch?v=gKbGIA7njQo](https://www.youtube.com/watch?v=gKbGIA7njQo)
> 
> [https://medium.com/developers-keep-learning/twitter-snowflake-approach-is-cool-3156f78017cb](https://medium.com/developers-keep-learning/twitter-snowflake-approach-is-cool-3156f78017cb)