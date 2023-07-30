---
layout: post
title: "Microservices vs Monolithic Architecture"
categories: springcloud
tags: springcloud
---

# Monolithic Architecture이란?

한 덩어리로 뭉쳐진 단일 서비스 개발 방식, 하나의 프로젝트로 구성되어 있으며 단일 패키지로 배포하는 아키텍처

소규모 프로젝트에 적합하고 대학교에서 하던 프로젝트 또는 스터디에서 진행하던 포트폴리오는 대부분 다음과 같은 방식으로 진행함

![monolithicArchitecture](/assets/postImages/MSAConcept/monolithicArchitecture.PNG)

## 장점

- 통합 시나리오 테스트가 수월하다
- 배포가 간단한다

## 단점

- 서비스가 커짐에 따라 빌드/테스트의 시간이 오래 걸림
- 개발 언어가 종속
  - Spring을 사용하면 Java, Nodejs를 사용하면 JS로 종속
- 선택적으로 확장이 불가능
  - 서비스마다 관여하는 비중이 크게 달라도 뭉쳐있기 때문에 전체가 확장
- 하나의 서비스가 다른 모든 서비스에 영향을 준다

# MSA(MicroService Architecture)란?

하나의 큰 어플리케이션을 여러 개의 작은 어플리케이션으로 쪼개어서 배포하는 아키텍처, 독립적인 기능을 수행하는 작은 단위의 서비스로 나누어 개발

대형 프로젝트에 적합하고, 트래픽을 많이 요구하는 곳에서 필요

![microserivcearchitecture](/assets/postImages/MSAConcept/microserivcearchitecture.PNG)

> API Gateway : 클라이언트와 웹서비스 사이에 중단 다리 역할을 한다.
- 공통 로직 구현, 서비스를 나누면 공통으로 로직을 짜야 하는 경우가 생긴다 ex) 인증, 인가
- 요청을 적절한 엔드포인트에 전달하고 트래픽 부하를 분산시킨다
- 요청 절차가 단순해짐, 클라이언트들은 여러 서비스에 요청을 진행해야되지만 그럴 필요가 없다

## 장점

- 서비스 별 배포가 가능
  - 배포 시에 전체 서비스 중단이 필요없다
  - CI / CD(지속적인 통합, 지속적인 배포)가 수월하다
- 특정 서비스에 확장에 용이
  - 뭉쳐있지 않기 때문에 서비스를 선택적으로 확장이 가능하다
- 장애 발생시 부분적인 대처 가능
  - 서비스가 분리되어 있어 부분적인 장애에 대해서 격리 및 대처가 수월하다

## 단점

- 데이터가 여러 서비스에 분산되어 있어 한번에 조회하기 어렵다
- 서비스 간 호출 시 API를 이용하기 때문에 통신 비용과, latency(지연 시간)이 증가
- 서비스가 분리되어 있어 트랜잭션과 테스트가 복잡하다

### References

- [모놀리틱 사진](https://microservices.io/patterns/monolithic.html)
- [MSA 사진](https://awesomeopensource.com/project/raycad/go-microservices)
