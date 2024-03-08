---
layout: post
title: Circuit Breaker Pattern
categories: cs
tags: cs
---

소프트웨어 시스템을 설계할 때, 외부 서비스 호출이나 데이터베이스 액세스와 같은 외부 리소스와의 상호작용은 불가피하게 발생한다. 외부 리소스에 대한 호출은 다양한 이유로 실패할 수 있으며, 이는 전체 시스템에 치명적인 영향을 미칠 수 있다. 이러한 문제를 해결하기 위해 'Circuit Breaker Pattern(서킷 브레이커 패턴)'이라는 소프트웨어 디자인 패턴이 개발되었다.

서킷 브레이커 패턴은 전기 회로의 서킷 브레이커와 유사한 개념으로, 시스템 간의 통신에서 발생할 수 있는 장애에 대응하는 데 사용된다. 이 패턴은 소프트웨어 시스템에서 외부 서비스 호출 또는 데이터베이스 액세스와 같은 외부 리소스에 대한 접근을 관리하고, 시스템의 안정성을 향상시키는 데 도움이 준다.

### 작동 원리

![circuit-breaker-pattern](/assets/postImages/CircuitBreakerPattern/circuit-breaker-pattern.png)

Closed 상태: 외부 서비스 호출이 정상적으로 처리되는 상태. Circuit Breaker는 호출을 수행하고 외부 서비스의 응답을 기다린다.

Open 상태: 외부 서비스 호출에 대한 회로가 열린 상태. 호출은 차단되고, 대체 동작이 실행된다. 일정 시간이 지나면 Circuit Breaker는 Half-Open 상태로 변경되고 일부 호출을 시도하여 서비스의 상태를 확인한다.

Half-Open 상태: 회로가 자동으로 닫힌 상태. 일부 호출만 다시 시도한다. 서비스의 상태를 확인하고 회로를 열거나 닫을 준비를 한다.

> [https://java-design-patterns.com/patterns/circuit-breaker/#programmatic-example](https://java-design-patterns.com/patterns/circuit-breaker/#programmatic-example)

### 왜 사용하는가?

서킷 브레이커 패턴은 외부 리소스에 대한 과도한 요청을 방지하여 시스템 과부하를 줄여주고 외부 리소스의 장애로부터 시스템을 보호하여 전체 시스템의 안정성과 신뢰성을 향상시킨다. 또한, 장애 발생 시 적절하게 대응하여 사용자 경험을 긍정적인 방향으로 유도하고 일정 시간 이후 자동으로 리소스에 대한 호출을 시도하여 회복할 수 있게 한다.

분산 시스템에서 특히 유용하며, 마이크로서비스 아키텍처와 같은 현대적인 소프트웨어 개발에 필수적인 디자인 패턴 중 하나이다. 외부 리소스에 대한 호출로 인한 장애를 효과적으로 관리하고 시스템의 안정성을 확보하는 데 도움이 된다.