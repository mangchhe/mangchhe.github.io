---
layout: post
title: Backend for Frontend in Microservices (BFF)
categories: architecture
tags: architecture
---

마이크로서비스 아키텍처가 도입되면서 각 서비스는 독립적으로 개발 및 배포가 가능해져 전체 시스템의 유연성과 확장성이 향상되었다. 하지만 이러한 분산 구조는 서비스 간의 통신 복잡도를 증가시켰고 필요한 데이터를 찾고 관리하는 일이 더 어려워졌다.

클라이언트가 각각의 마이크로서비스에 다이렉트로 접근하는 것은 데이터 관리 및 통합 측면에서 비효율적이다.

<p align="center">
    <img src="/assets/postImages/BackendForFrontedInMicroservices/bff.png" width="550">
</p>

BFF는 복잡해진 마이크로서비스 간의 통신을 단순화하고, 데이터를 중앙에서 통합 관리한다. 이는 클라이언트에게 최적화되고 단순하면서도 효율적인 API 접근을 가능하게 하며 불필요한 네트워크 통신을 줄이고 캐싱을 통해 애플리케이션의 성능을 향상시킬 수 있다.

하지만 BFF를 도입하면 시스템의 구조가 복잡해질 수 있으며 잘못 구현된 경우 오히려 통신 및 리소스 낭비를 초래할 수 있다. 따라서 현 시스템에서 BFF가 필요한지 잘 판단하여 도입해야 한다.

> [https://dev.to/adelhamad/bff-backend-for-frontend-design-pattern-with-nextjs-3od0](https://dev.to/adelhamad/bff-backend-for-frontend-design-pattern-with-nextjs-3od0)