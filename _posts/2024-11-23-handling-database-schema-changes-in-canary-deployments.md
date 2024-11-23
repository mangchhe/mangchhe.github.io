---
layout: post
title: Handling Database Schema Changes in Canary Deployments
categories: cs
tags: cs, canary, deployment, database
---

롤링 또는 카나리 배포 시, 두 버전이 공존하게 되므로 이전 버전과 현재 버전의 **호환성**을 반드시 고려해야 한다. 

특히 **DB 스키마 변경**은 데이터 무결성과 서비스 안정성을 보장하기 위해 더 신중한 접근이 필요하다.

##### 대응 방안

**1)** 새로운 컬럼(new)을 추가하고, 기존 컬럼(old)과 추가로 새로운 컬럼(new)에 데이터를 쓰는 애플리케이션을 배포한다. 데이터 읽기는 기존 컬럼(old)을 유지한다.

**2)** 배포 이후 새로운 컬럼(new)에 데이터가 정상적으로 기록되고, 서비스가 안정적이라면 기존 컬럼(old)의 데이터를 새로운 컬럼으로 마이그레이션한다.

**3)** 데이터 마이그레이션이 완료되면, 데이터 읽기 로직을 새로운 컬럼(new)으로 전환하여 애플리케이션을 배포한다.

**4)** 서비스 안정성을 확인한 뒤, 기존 컬럼(old)을 제거한다.

> [https://mangchhe.github.io/devops/2021/08/04/ZeroDowntimeDeployment/](https://mangchhe.github.io/devops/2021/08/04/ZeroDowntimeDeployment/)
> 
> [https://www.sisson.ru/changing-column-data-with-canary-release-migration-strategy.php](https://www.sisson.ru/changing-column-data-with-canary-release-migration-strategy.php)