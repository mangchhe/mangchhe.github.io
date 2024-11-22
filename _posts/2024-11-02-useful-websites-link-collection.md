---
layout: post
title: Useful Websites Link Collection
categories: etc
tags: etc
---

##### OOP

- [일급 컬렉션 (First Class Collection)의 소개와 써야할 이유](https://jojoldu.tistory.com/412)

##### Java

- [도움이 될수도 있는 JVM memory leak 이야기](https://techblog.woowahan.com/2628/)
  - RestTemplate, Zuul, neflix.servo metrics가 url pattern을 고유한 키로 판단하면서 발생하는 memory leak, ngrinder
- [Java의 미래, Virtual Thread](https://techblog.woowahan.com/15398/?fbclid=IwAR3DtFiX-vOdcMc1wlNPOESBZukFXGiPXzlMj7Ip7MGm6m9QCHhH2Yl0H5s)
  - JDK21, 함수의 색 문제, Coroutine, CPU bound 작업엔 비효율적, 수시로 생성 및 소멸하기 때문에 thread local은 작게 유지

##### Spring

- [Spring Boot에서 S3에 파일을 업로드하는 세 가지 방법](https://techblog.woowahan.com/11392)
  - Stream, MultipartFile, Aws Multipart Upload
- [정산지기를 향한 첫걸음 (feat. 파일럿 프로젝트)](https://techblog.woowahan.com/2668/)
  - `@Where(clause = "delete = 0")`, totalCount 캐싱, 권한 관리, chunkSize, 코드 리뷰
- [멀티모듈 설계 이야기 with Spring, Gradle](https://techblog.woowahan.com/2637/)
- [우아한 feign 적용기](https://techblog.woowahan.com/2630/)
- [OpenAPI Specification을 이용한 더욱 효과적인 API 문서화](https://tech.kakaopay.com/post/openapi-documentation/)
- [스프링 배치(Spring Batch) 전환기](https://blog.naver.com/tmondev/220772936562)
- [풀필먼트 입고 서비스팀에서 분산락을 사용하는 방법 - Spring Redisson](https://helloworld.kurly.com/blog/distributed-redisson-lock/)
- [Validation 어디까지 해봤니?](https://meetup.nhncloud.com/posts/223)

##### Kotlin

- [Kotlin으로 Spring AOP 극복하기!](https://tech.kakaopay.com/post/overcome-spring-aop-with-kotlin/)
  - Trailing Lambdas, Reverse Argument, Self Invocation

##### JPA

- [응? 이게 왜 롤백되는거지?](https://techblog.woowahan.com/2606/)
  - Propagation, Transaction, Rollback, Try-Catch
 
##### DB

- [캐시 문제 해결 가이드 - DB 과부하 방지 실전 팁](https://toss.tech/article/cache-traffic-tip)
  - Cache Stampede, Jitter, Cahce Penetration, Null Object Pattern, Failover, Hotkey, Distributed Lock for cache hit
- [인덱스로 성능 개선하기 (feat. B-Tree 인덱스)](https://jminc00.tistory.com/87)
- [NOT IN 대신 JOIN을 통한 쿼리 최적화 방법 (쿼리 시간 96% 절감 후기)](https://medium.com/iotrustlab/sql-optimization-from-not-in-to-left-join-16132ca048ba)

##### Log

- [토스ㅣSLASH 23 - 분산 추적 체계 & 로그 중심으로 Observability 확보하기](https://youtu.be/Ifz0LsfAG94?feature=shared)
  - L7 Load Balancer, X-Forwarded-For, Proxy Protocol, Global Trace Id, MDC, Istio

##### DevOps

- [좌충우돌 Terraform 입문기](https://techblog.woowahan.com/2646/)
  - cycle error, aws_sqs_queue_policy
- [Datadog - "저는 모니터링 요정입니다. ✋🏻"](https://oliveyoung.tech/blog/2022-06-22/How-to-Set-up-Build-ECS-Fargate-And-Datadog/)

##### Architecture

- [우리동네GS BFF 구현기 Step 1 - 도입 배경과 설계](https://gsretail.tistory.com/40)

##### Culture

- [공통시스템개발팀 코드 리뷰 문화 개선 이야기](https://techblog.woowahan.com/7152/)
  - D-n 규칙, Pn 규칙, pre-commit
- [우린 Git-flow를 사용하고 있어요](https://techblog.woowahan.com/2553/)

##### Fuction

- [매일 배포하는 팀이 되는 여정(2) — Feature Toggle 활용하기](https://medium.com/daangn/매일-배포하는-팀이-되는-여정-2-feature-toggle-활용하기-b52c4a1810cd)

##### Performance

- [채널톡 실시간 채팅 서버 개선 여정 - 1편 : 레디스의 'Pub/Sub'](https://channel.io/ko/blog/real-time-chat-server-1-redis-pub-sub)
  - Redis, Socket.io, broadcast - O(NM)
- [WMS 재고 이관을 위한 분산 락 사용기](https://techblog.woowahan.com/17416/)
- [대량의 트래픽이 몰려올 때 나는 어떻게 해야하나? - (feat. Cache, 대기열 구현)](https://velog.io/@joshuara7235/대량의-트래픽이-몰려올-때-나는-어떻게-해야하나-feat.-Cache-대기열-구현)

##### Chat

- [라이브채팅 플랫폼 구현기 1탄 : 개발 언어 및 기반기술 조사](https://kakaoentertainment-tech.tistory.com/109)
  - HTTP Polling, Webscoket, MongoDB, Kafka, Trie, Aho-Corasick
- [라이브채팅 플랫폼 구현기 2탄 : 아키텍처 및 성능 테스트](https://kakaoentertainment-tech.tistory.com/110)
  - 테스트 플랫폼 비교 및 분석, Consistent Hashing

##### Helpful Things

- [넥스터즈 지원자 서류 검토 후기](https://imksh.com/108)
- [개발자 이력서 작성 및 변화 과정 (이력서 공개)](https://imksh.com/120)
- [컬리의 새로운 배송 시스템 구축 과정과 우리가 배운점](https://helloworld.kurly.com/blog/2023-delivery-system/)

##### Tools

- [IntelliJ 등에서 메모리 점유율이 계속 높아질 때 (feat. vanilla JBR)](https://jojoldu.tistory.com/800)