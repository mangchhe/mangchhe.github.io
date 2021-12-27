---
title: "[Spring Cloud] Configuration 정보 갱신 방법 With Bus"
description: 이전 포스터에서 Spring Cloud Config를 이용하여 설정 파일들을 따로 관리하는 서버를 구축하였다. 이때 Config Server에 있는 Configuration 정보들이 변경이 이루어지게 되면 변경된 내용들을 어떻게 마이크로서비스들에게 전달할 수 있는지에 대해서 actuator, Spring Cloud Bus에 대해서 배워보려고 한다.
categories:
 - SpringCloud
tags:
 - Spring Cloud
 - MSA
 - Spring Cloud Bus
 - RabbitMQ
 - AMQP
---

# Configuration 갱신 방법

<hr>

1. 서비스 재기동
2. Actuator refresh
3. Spring Cloud Bus

소개 할 갱신 방법에는 위와 같이 총 세 가지가 존재하는데 서비스 재기동 방식에 대해서는 따로 설명하지 않고 Actuator refresh 하는 방법부터 설명하려고 한다.

# Actuator Refresh

<hr>

Spring Boot 에서 자체적으로 제공하는 actuator를 이용하려고 한 actuator는 애플리케이션을 모니터링하고 관리하는데 필요한 여러 기능들이 포함되어있다.

우리는 여러 기능 중에서 Configuration 정보를 refresh하는 방법에 대해서 알아보려고 한다. 이 외 기능들은 [Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html)를 참고하면 된다.

## 의존성 추가

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

## application.yml 수정

``` yml
management:
  endpoints:
    web:
      exposure:
        include: refresh
```

## Endpoint 호출

앞서 설정이 끝났다면 Config Server에서 수정된 Configuration 정보를 재기동 없이 적용시키기 위해서 `<호스트 네임>:포트/actuator/refresh` 엔드 포인트를 POST 방식으로 호출하게 되면 변경된 내용이 적용될 것이다.

## actuator 만으로는 부족한 이유

재기동 없이 Configuration 정보를 수정은 가능해졌지만 Configuration 수정이 이루어지기 위해서는 해당 서비스의 엔드 포인트를 호출해야 하는 단점이 있다. 만약 변경이 일어난 서비스가 20개라고 한다면 20개의 엔드 포인트를 모두 호출해야되기 때문에 actuator를 이용하는 방식도 효율적이라고 생각할 수 는 없다.

# Spring Cloud Bus

<hr>

`RabbitMQ`나 `Kafka`와 같은 메시지 브로커를 이용하여 변경사항들을 각 서비스에 전달하고 Spring Cloud Bus는 메시지 브로커들과 서비스를 연결하는 역할을 하게 된다. 이러한 방법을 이용하게 되면 앞서 이용했던 actuator의 단점을 해소할 수 있다.

## RabbitMQ 설치(Window)

### [Erlang](https://www.erlang.org/downloads) 설치

RabbitMQ를 사용하려면 Erlang을 설치해야한다.

![erlangInstall](/assets/postImages/ConfigRefreshMethod/erlangInstall.PNG)

### [RabbitMQ](https://www.rabbitmq.com/install-windows.html#installer) 설치

![RabbitMQInstall](/assets/postImages/ConfigRefreshMethod/RabbitMQInstall.PNG)

### Erlang, RabbitMQ 환경 변수 설정

설치 후에 환경 변수가 자동으로 설정되어 있지 않다면 PATH에 맞게 직접 추가해준다.

![erlangRabbitMQPath](/assets/postImages/ConfigRefreshMethod/erlangRabbitMQPath.PNG)

### RabbitMQ Plugin 설치

``` bash
rabbitmq-plugins enable rabbitmq_management
```

### RabbitMQ 실행 & 접속

![RabbitMQServiceRun](/assets/postImages/ConfigRefreshMethod/RabbitMQServiceRun.PNG)

윈도우 키를 눌러서 서비스를 검색하고 RabbitMQ가 정상적으로 실행이 되어 있는지 확인한다.

![RabbitMqConnect](/assets/postImages/ConfigRefreshMethod/RabbitMqConnect.PNG)

http://localhost:15672/ 로 접속하게 되면 위와 같이 username, password를 입력할 수 있는데 default 값으로 guest, guest로 설정되어있어 입력하면 된다.

![RabbitMqConnect2](/assets/postImages/ConfigRefreshMethod/RabbitMqConnect2.PNG)

로그인하고 다음과 같은 창을 만나게 되면 준비는 끝났다고 볼수 있다.

## ConfigService

Spring Cloud Bus를 적용하기 위해서 Config Service에 적용시켜야 될 내용들에 대해서 알아보자

### 의존성 추가

``` java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

### application.yml 수정

``` yml
spring:
  application:
    name: <application-name>
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest

management:
  endpoints:
    web:
      exposure:
        include: busrefresh
```

rabbitmq 정보들에 대해서 기입하고 actuator 기능에 busrefresh를 추가한다.

## MicroServices

Spring Cloud Bus를 적용하기 위해서 MicroServices에 적용시켜야 될 내용들에 대해서 알아보자

### 의존성 추가

``` java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

### application.yml 수정

``` yml
spring:
  application:
    name: <application-name>
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest

management:
  endpoints:
    web:
      exposure:
        include: busrefresh
```

rabbitmq 정보들에 대해서 기입하고 actuator 기능에 busrefresh를 추가한다.

## ConfigService, MicroServices 구동

![RabbitMqConnect3](/assets/postImages/ConfigRefreshMethod/RabbitMqConnect3.PNG)

구동을 하고 RabbitMQ gui에 들어가서 Connections를 확인하게 되면 다음과 같이 구동하고 연결된 서비스들에 대한 상태들이 나타나게 된다.

## Endpoint 호출

rabbitmq에 연결되어 있는 `<호스트 네임>:포트/actuator/busrefresh` 엔드 포인트를 POST 요청을 하게 되면 rabbitmq에 연결되어 있는 모든 서비스에 변경된 Configuration 정보들이 전달되는 것을 확인할 수 있다.
