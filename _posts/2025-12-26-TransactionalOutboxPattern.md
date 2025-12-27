---
layout: post
title: "Transactional Outbox Pattern"
categories: architecture
tags: kafka, cs
---

마이크로서비스에서 Kafka/SQS 같은 비동기 메시징을 도입하면 서비스 간 결합을 낮추고 확장성을 얻는 대신, **데이터 정합성** 문제가 자주 튀어나온다.

대표적으로 아래가 반복된다.

- 발행돼야 하는 이벤트가 발행되지 않음(유실)
- 발행되면 안 되는 이벤트가 발행됨(유령 이벤트)
- 배포/장애 타이밍에 간헐적으로 누락/지연이 발생함
- 같은 이벤트가 중복 발행될 수 있음(at-least-once)

이 글에서는 위 문제를 실무적으로 해결하기 위한 패턴인 **Transactional Outbox Pattern(트랜잭셔널 아웃박스)**를 정리한다.

#### 적용 범위

아래 중 하나라도 해당되면 outbox 적용을 먼저 고민하는 편이 안전하다.

- 이벤트 유실이 비즈니스 장애로 직결됨
  - 예: 주문 완료 이후 정산/배송/알림 등 후속 처리
- 동기 호출(HTTP)로 묶기엔 결합/지연이 부담됨
- 배포/장애 타이밍의 간헐 누락이 반복적으로 문제를 만든 적이 있음

반대로 통계/로그성 이벤트처럼 "누락돼도 사람이 복구 가능한 영역"이면, 비용 대비 과한 선택일 수 있다.

#### 문제

핵심은 메시지 브로커 발행과 DB 트랜잭션 커밋이 **서로 다른 시스템**이라는 점이다.

- 트랜잭션 밖 publish
  - 도메인은 커밋됐는데 publish가 실패하면 이벤트 유실
- 트랜잭션 안 publish
  - publish는 됐는데 commit이 실패하면 유령 이벤트
  - commit 지연 시점엔 컨슈머가 조회하면 아직 상태가 반영되지 않을 수도 있음

#### 핵심 아이디어

outbox는 "브로커에 잘 쏘기"가 아니라, **발행 요청을 DB 트랜잭션에 묶어 기록**하는 패턴이다.

- 같은 트랜잭션에서
  - 도메인 변경
  - outbox row insert(= 발행할 이벤트 기록)
- 이후 릴레이가 outbox를 읽어 publish
- 실패하면 재시도 → 결국 **eventual consistency**

#### 전체 흐름

![outbox-flow](/assets/postImages/TransactionalOutboxPattern/outbox-flow.png){: width="700" }

#### 구현 방식 1: Polling Relay(배치/워커)로 발행

- 서비스 트랜잭션에서는 outbox에만 저장
- 릴레이(배치/워커/크론)가 outbox를 polling 하며 발행

##### Producer

```java
@Transactional
public void placeOrder(PlaceOrderCommand cmd) {
  orderRepository.save(order);
  outboxRepository.save(OutboxEvent.of("OrderPlaced", payload));
}
```

##### Relay

```java
@Scheduled(fixedDelay = 1000)
public void relay() {
  List<OutboxEvent> events = outboxRepository.findUnpublished(100);
  kafkaProducer.send(events);
  outboxRepository.markPublished(events); // 또는 delete
}
```

- 운영이 단순함
- polling 주기만큼 발행 지연이 생길 수 있음

#### 구현 방식 2: TransactionalEventListener로 BEFORE/AFTER 훅 분리

스프링을 쓰면 "커밋 전/후"를 훅으로 분리할 수 있다.

- **BEFORE_COMMIT**: outbox 기록(같은 트랜잭션)
- **AFTER_COMMIT**: 브로커 발행(커밋 확정 이후)

- "이벤트 이후 처리 로직은 리스너에 모은다"는 일관성을 만들기 좋다.

![outbox-transactional-event-listener](/assets/postImages/TransactionalOutboxPattern/outbox-transactional-event-listener.png){: width="700" }

#### 운영 체크리스트

outbox는 구현보다 운영 디테일에서 품질이 갈린다.

- 상태 모델
  - `published=true/false`만 두면 원인 파악이 어려워진다.
  - 최소 `init` / `send_success` / `send_fail`로 나누는 편이 낫다.
  - `init`이 오래 남으면 "발행 로직이 아예 실행되지 못한 케이스"를 의심하기 좋다.
- 배포/종료(graceful shutdown)
  - AFTER_COMMIT + `@Async` 조합은 배포 타이밍에 작업 유실이 날 수 있다.
  - 비동기 executor graceful shutdown 설정을 점검한다.

```java
executor.setWaitForTasksToCompleteOnShutdown(true); // 종료 시 실행 중인 작업이 끝날 때까지 기다림
executor.setAwaitTerminationSeconds(10); // 최대 대기 시간(초)
```
- 재시도 전략
  - 주기/limit/backoff, 재시도 대상 조건(예: 생성 후 N분 경과)과 알람 조건을 정한다.
- Consumer 멱등
  - outbox는 보통 **at-least-once delivery**라 중복 발행이 가능하다.
  - message id 기반 중복 제거 또는 upsert/버전 기반 수렴 설계가 필요하다.
- 보관 정책
  - `send_success`를 남기면 추적/감사/리플레이에 유리하다.
  - 대신 TTL/아카이빙/파티셔닝 같은 정리 전략을 같이 가져간다.

#### 정리

outbox의 핵심은 "도메인 변경"과 "이벤트 발행 요청 기록(outbox)"을 **같은 DB 트랜잭션에 묶는 것**이다.

그리고 운영에서 체감하는 품질(안정성/발행 지연)은 결국 아래에서 갈린다.

- 릴레이(워커) 처리량/주기(= 발행 지연)
- 상태 모델 + 재시도/알람(= 유실/장애 대응)
- 배포/종료(graceful shutdown)
- 컨슈머 멱등

#### References

- [트랜잭셔널 아웃박스 패턴의 실제 구현 사례 (29CM)](https://medium.com/@greg.shiny82/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%94%EB%84%90-%EC%95%84%EC%9B%83%EB%B0%95%EC%8A%A4-%ED%8C%A8%ED%84%B4%EC%9D%98-%EC%8B%A4%EC%A0%9C-%EA%B5%AC%ED%98%84-%EC%82%AC%EB%A1%80-29cm-0f822fc23edb)
- [분산 시스템에서 메시지 안전하게 다루기](https://blog.gangnamunni.com/post/transactional-outbox)
- [Pattern: Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html)
