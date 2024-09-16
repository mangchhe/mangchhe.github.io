---
layout: post
title: "Infrastructure as Code: Principles, Pattenrns"
categories: devops
tags: devops
---

> [https://www.cloudknit.io/blog/principles-patterns-and-practices-for-effective-infrastructure-as-code](https://www.cloudknit.io/blog/principles-patterns-and-practices-for-effective-infrastructure-as-code)

IaC는 소프트웨어 시스템에서 사용되는 코딩 기술을 인프라 확장하는 접근 방식이다. 이는 신속하고 안정적인 시스템 구축을 지원하는 Deveops 방식 중 하나이다.

애플리케이션에 대한 Continuous Delivery를 달성하려면, **빠르고 안정적인 프로비저닝 메커니즘**을 갖추는 것이 중요하다.

> [Continuous Delivery](https://martinfowler.com/bliki/ContinuousDelivery.html)
>
> Continuous Delivery is a software development discipline where you build software in such a way that the software can be released to production at any time.

#### 주요 원칙

- 멱등성 (Idempotency) : 프로비저닝을 단순화하고 일관되지 않은 결과가 발생할 가능성을 줄인다.
- 불변성 (Immutability) : 기존 인프라에서의 변경이 아닌 새 인프라로 교체하여 Configuration drift 문제를 해결한다.

> [Configuration Drift](http://kief.com/configuration-drift.html)
>
> Configuration Drift is the phenomenon where servers in an infrastructure become more and more different from one another as time goes on, due to manual ad-hoc changes and updates, and general entropy.

#### 패턴과 관행

- 소스 제어 : 모든 코드는 VCS에 관리되어야 하며, 스크립트나 배포 파이프라인도 포함된다. 이는 코드 변경 이력을 추적하고, 문제 발생 시 원인을 파악하는 데 필수적이다.
- 모듈화와 버전 관리 : 모듈화하며 유지 관리가 용이하며, 독립적으로 배포할 수 있는 작은 단위로 나눌 수 있다.
- 문서화 : 광범위한 문서가 필요하진 않지만, 필수적인 문서는 항상 유지해야 한다. 코드와 가까운 위치에 문서를 배치하는 것이 좋다. 예를 들어 같은 레포에 README가 있다.
- 테스트 : IaC에서도 정적 분석, 통합 테스트, 스모크 테스트 등 여러 수준의 테스트가 필요하고 테스트 피라미드를 통해 비용과 시간을 최소화하면서 피드백 속도를 높이는 것이 중요하다.
- 자동화 : IaC 파이프라인을 통해 인프라를 프로비저닝하고, GitOps를 통해 코드 변경 사항을 쉽게 관리하며, 지속적인 모니터링과 자동화를 구현할 수 있다.

이러한 원칙과 패턴을 통해 IaC의 안정성과 가시성을 크게 향상시키고 유지 보수를 용이하게 해준다.