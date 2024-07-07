---
layout: post
title: Trunk Based Development (TBD)
categories: git
tags: git
---

> [https://trunkbaseddevelopment.com/](https://trunkbaseddevelopment.com/)

트렁크 기반 개발(TBD)은 개발자들이 mainline(main or trunk)이라는 단일 브랜치에서 협업하는 브랜치 모델이다. 이 모델은 장기적으로 운용되는 브랜치를 피하고, 작은 커밋을 자주 반영하여 머지 헬을 피하고 빌드를 깨뜨리지 않는다.

![image](https://github.com/mangchhe/mangchhe.github.io/assets/50051656/54d19b37-2190-4554-9452-04baa4f938bb)

TBD는 기본적으로 작은 팀을 위한 모델과 확장된 모델로 나눌 수 있다. 작은 팀을 위한 TBD는 mainline에 실시간으로 다이렉트로 커밋을 하는 반면, 확장된 TBD는 기능 단위로 브랜치를 생성하여 PR을 통해 코드 리뷰와 빌드 자동화를 수행한다. 두 전략은 팀의 규모와 커밋 빈도에 따라 선택하여 진행하면 된다.

TBD는 지속적인 통합(CI) 및 지속적인 배포(CD)가 필수적인 요소이다. 개발자들은 하루에 적어도 한 번 이상 커밋하여 코드 베이스가 항상 배포 가능하도록 유지하며 개발을 진행한다.

따라서 TBD를 채택하면 팀 내에 코드 통합의 복잡성을 줄일 수 있고, 빠르고 안정적인 배포를 통해 제품의 품질을 향상시킬 수 있다. 또한, 규모에 맞는 효율적인 개발 프로세스를 유지할 수 있어 개발 생산성과 협업 효율성이 크게 기여할 수 있다.