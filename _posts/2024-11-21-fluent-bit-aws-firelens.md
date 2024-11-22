---
layout: post
title: Fluent Bit, AWS FireLens
categories: devops
tags: devops
---

> [AWS FireLens](https://aws.amazon.com/ko/about-aws/whats-new/2019/11/aws-launches-firelens-log-router-for-amazon-ecs-and-aws-fargate/)

AWS FireLens는 **Amazon ECS** 및 **AWS Fargate** 컨테이너에서 생성된 로그를 처리하고 외부 서비스로 전달하기 위한 **통합 로그 라우터**이다. 

FireLens는 Fluent Bit 또는 Fluentd와 통합되어 동작하며, 컨테이너 로그를 손쉽게 관리하고 다양한 목적지로 전송할 수 있는 환경을 제공한다.

> [Fluent Bit](https://github.com/fluent/fluent-bit)

Fluent Bit은 오픈소스 **경량 로그 프로세서**로, FireLens가 로그를 처리하기 위해 기본적으로 사용하는 도구이다.

리소스를 적게 사용하면서도 강력한 로그 처리 기능을 제공하며, AWS FireLens와 함께 사용하면 컨테이너 환경에서 손쉽게 로그를 처리하고 전송할 수 있다.

##### FireLens와 Fluent Bit의 관계

FireLens는 AWS ECS 및 Fargate 환경에서 Fluent Bit(또는 Fluentd)을 실행하고 관리하는 인터페이스이다. 

즉, FireLens는 Fluent Bit의 실행 환경을 제공하며, Fluent Bit은 로그 처리와 전송을 담당한다.

이 조합을 통해 로그의 수집, 필터링, 변환, 전송 과정을 단순화하고 강력한 제어 기능을 제공받을 수 있다.

> [https://mangchhe.github.io/devops/2024/11/20/datadog-integration-with-aws-ecs-faragate-using-terraform/](http://localhost:4000/devops/2024/11/20/datadog-integration-with-aws-ecs-faragate-using-terraform/)
> 
>[https://docs.datadoghq.com/ko/integrations/ecs_fargate/?tab=webui#fluent-bit-%EB%B0%8F-firelens](https://docs.datadoghq.com/ko/integrations/ecs_fargate/?tab=webui#fluent-bit-%EB%B0%8F-firelens)