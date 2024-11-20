---
layout: post
title: Datadog Integration with AWS ECS Fargate using Terraform
categories: devops
tags: devops
---

AWS ECS Fargate 기반 애플리케이션에서 Datadog을 활용하여 로그와 APM을 통합하는 방법

**Datadog Agent 컨테이너**
- APM 데이터 및 메트릭 수집
- Java Agent를 통해 분산 추적 및 성능 모니터링 지원

**애플리케이션 컨테이너**
- FireLens, Fluent Bit를 사용하여 Datadog의 로그를 수집하여 Datadog으로 직접 전송

#### Datadog Agent 태스크 정의

```terraform
resource "aws_ecs_task_definition" "task_definition" {
  family                   = var.family
  network_mode             = var.network_mode
  cpu                      = var.cpu
  memory                   = var.memory
  requires_compatibilities = var.requires_compatibilities
  task_role_arn            = var.task_role_arn
  execution_role_arn       = var.execution_role_arn

  container_definitions = jsonencode([
    {
      name      = "datadog-agent"
      image     = "public.ecr.aws/datadog/agent:latest"
      cpu       = var.cpu
      memory    = var.memory
      essential = true
      portMappings : [
        {
          hostPort      = var.datadog_agent_port
          protocol      = "tcp"
          containerPort = var.datadog_agent_port
        }
      ]
      environment = [
        {
          name  = "DD_APM_ENABLED"
          value = "true"
        },
        {
          name  = "DD_API_KEY"
          value = var.datadog_api_key
        },
        {
          name  = "DD_SITE"
          value = "datadoghq.com"
        },
        {
          name  = "ECS_FARGATE"
          value = "true"
        }
      ]
      logConfiguration = {
        logDriver = "awslogs"
        options = {
          awslogs-group         = "/ecs/datadog-agent"
          awslogs-region        = "ap-northeast-2"
          awslogs-stream-prefix = "datadog-agent"
        }
      }
    }
  ])
}
```

#### 애플리케이션 태스크 정의

- Datadog Java Agent를 사용해 로그와 추적 데이터를 Datadog으로 전송하며, FireLens를 활용해 로그를 처리한다.

```terraform
resource "aws_ecs_task_definition" "task_definition" {
  family                   = var.family
  network_mode             = var.network_mode
  cpu                      = var.cpu
  memory                   = var.memory
  requires_compatibilities = var.requires_compatibilities
  task_role_arn            = var.task_role_arn
  execution_role_arn       = var.execution_role_arn

  runtime_platform {
    operating_system_family = "LINUX"
    cpu_architecture        = "X86_64"
  }

  container_definitions = jsonencode([
    {
      name      = var.container_name
      image     = var.image
      cpu       = var.cpu
      memory    = var.memory
      essential = true
      logConfiguration = {
        logDriver = "awsfirelens"
        options = {
          Name       = "datadog"
          Host       = "http-intake.logs.datadoghq.com"
          TLS        = "on"
          apikey     = var.datadog_api_key
          dd_source  = "ecs"
          dd_tags    = "env:${var.environment},team:plot-dev"
          dd_service = var.datadog_service_name
          dd_message_key = "log"
          provider   = "ecs"
        }
      }
      portMappings = [
        {
          containerPort = var.container_port
          protocol      = "tcp"
        }
      ]
    },
    {
      name      = "firelens-container"
      image     = "amazon/aws-for-fluent-bit:latest"
      essential = false
      firelensConfiguration = {
        type = "fluentbit"
        options = {
          enable-ecs-log-metadata = "true"
        }
      }
      logConfiguration = {
        logDriver = "awslogs"
        options = {
          awslogs-group         = "/ecs/firelens-container"
          awslogs-region        = "ap-northeast-2"
          awslogs-stream-prefix = "firelens"
        }
      }
    }
  ])
}
```

#### Service Discovery 설정

- ECS 클러스터 내에서 Datadog Java Agent가 실행 중인 Datadog Agent 컨테이너를 동적으로 식별할 수 있도록 설정한다.

```terraform 
resource "aws_service_discovery_private_dns_namespace" "namespace" {
  name        = "${var.environment}.local"
  description = "Private DNS namespace for Datadog Agent"
  vpc         = var.vpc_id
}

resource "aws_service_discovery_service" "datadog" {
  name = "datadog"

  dns_config {
    namespace_id = aws_service_discovery_private_dns_namespace.namespace.id

    dns_records {
      ttl  = 10
      type = "A"
    }

    routing_policy = "MULTIVALUE"
  }

  health_check_custom_config {
    failure_threshold = 1
  }
}

resource "aws_ecs_service" "datadog" {
  ...

  service_registries {
    registry_arn = var.service_discovery_service_backend_arn
  }
}
```

#### Datadog Java Agent를 활용한 APM 설정

```
// Dockerfile
ADD https://dtdg.co/latest-java-tracer /usr/agent/dd-java-agent.jar

ARG JAR_FILE=build/libs/app.jar
COPY ${JAR_FILE} app.jar

ENTRYPOINT java -javaagent:/usr/agent/dd-java-agent.jar \
    -Ddd.agent.host=datadog.${PROFILE}.local \
    -Ddd.profiling.enabled=false \
    -XX:FlightRecorderOptions=stackdepth=256 \
    -Ddd.logs.injection=true \
    -Ddd.service=app-api \
    -Ddd.env=${PROFILE} \
    -Dspring.profiles.active=${PROFILE} \
    -jar /app.jar
```

>[https://docs.datadoghq.com/ko/integrations/ecs_fargate/?tab=webui#fluent-bit-%EB%B0%8F-firelens](https://docs.datadoghq.com/ko/integrations/ecs_fargate/?tab=webui#fluent-bit-%EB%B0%8F-firelens)
> 
> [https://oliveyoung.tech/blog/2022-06-22/How-to-Set-up-Build-ECS-Fargate-And-Datadog/](https://oliveyoung.tech/blog/2022-06-22/How-to-Set-up-Build-ECS-Fargate-And-Datadog/)