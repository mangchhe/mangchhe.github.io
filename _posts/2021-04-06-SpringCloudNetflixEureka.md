---
title: "[Spring Cloud] Netflix Eureka에 대해서 실습을 통해 알아보자"
description: 서비스들이 동적으로 확장, 축소가 이루어지는 것을 관리하는 위한 서버인 Eureka(Service Discovery Server)에 대해서 배워보자
categories:
 - SpringCloud
tags:
 - Spring Cloud
 - MSA
 - Eureka
 - Service Discovery
---

# Spring Cloud Netflix Eureka란?

<hr>

MSA는 여러 개의 서비스를 한곳에 모아 패키징하여 관리하는 방식이 아닌 각 서비스들을 독립적으로 관리하고 유기적으로 조립시킬 수 있는 방식이다

이때 각 서비스들은 서로 다른 호스트 네임과 포트들을 가지게 되는데 이것을 개별적으로 하드코딩하여 관리하면 IP가 동적으로 변경되는 일이 많은 클라우드 환경에서는 유지보수에 큰 어려움을 겪게 될 것이다

`Eureka`는 이러한 서비스들을 한대 모아 관리하고 각 서비스들의 호스트 이름과 포트를 하드 코딩하지 않고도 서비스 서로를 찾아 통신할 수 있게 해준다

![eureka](/assets/postImages/SpringCloudNetflixEureka/eureka.PNG)

위 그림에서 서로 다른 서비스 인스턴스들에 정보에 대해서 `Eureka(ServiceDiscovery)`에 등록하게 되면 클라이언트가 요청 정보를 API Gateway에 전달하게 되면 `ServiceDiscovery`에게 요청 정보가 어느 서비스에 있는지 물어보게 되고 등록되어 있는 서비스들 중에 찾아 `API Gateway`에 전달해주면 해당하는 인스턴스에게 값을 요청하고 클라이언트에게 다시 응답이 전달되게 된다

1. 클라이언트 요청
2. `API Gateway`에서 `ServiceDiscovery`에 해당 요청 정보 위치 물어봄
3. `ServiceDiscovery`가 `API Gateway`에 정보 전달
4. `API Gateway`는 정보를 받아 해당 서비스에 요청
5. 클라이언트에게 응답

# Spring Cloud Netflix Eureka 실습

<hr>

실제 코드를 작성을 통해 `Eureka`에 대해서 알아보자

**예제 소스 파일** : [Github](https://github.com/mangchhe/WEB_Cloud_Tutorial)

## Eureka Server

### 개발 환경

- SpringBoot 2.4.4
- Maven
- Java 11
- SpringCloud 2020.0.2

> #### 의존성(dependency)

``` xml
<properties>
    <spring-cloud.version>2020.0.2</spring-cloud.version>
</properties>
<dependencies>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

프로젝트 생성시에 Eureka Server를 선택하여 추가해주었다면 pom.xml 파일안에 위와 같이 의존성이 등록되어 있는 것을 확인할 수 있다
만약에 없다면 수동으로 추가해주면 된다

### 설정 파일 수정(application.yml)

``` yml
server:
  port: 8761

spring:
  application:
    name: discoveryservice

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

위와 같이 사용할 포트를 설정해주고 각각의 `MicroSerivce` 고유 아이디인 이름을 지정해주어야 한다
`register-with-eureka`과 `fetch-registry`를 false로 설정해주어야 한다
이유는 eureka 라이브러리가 포함된채 어플리케이션을 가동시키면 위에 두 설정이 true로 되서 클라이언트 역할로 전화부에 등록하듯이 자신의 정보를 자신에게 등록하는 현상이되어 의미 없는 작업이다

### Eureka 서버 실행

``` java
@SpringBootApplication
@EnableEurekaServer
public class EcommerceApplication {

    public static void main(String[] args) {
        SpringApplication.run(EcommerceApplication.class, args);
    }

}
```

어플리케이션을 실행시에 처음 실행 되는 자바 코드 파일에 가서 `@EnableEurekaServer` 어노테이션을 달아줘야 Eureka 서버가 정상적으로 구동되는 것을 확인할 수 있다
`localhost:8761`로 접속하게 되면 아래와 대시보드 화면이 나타나는 것을 확인할 수 있다

![eurekaServer](/assets/postImages/SpringCloudNetflixEureka/eurekaServer.PNG)

이 대시보드는 유레카 서버가 제공하는 기본적인 정보들을 담고 있다
서버가 언제 가동이 되었는지, 나에게 등록된 인스턴스(MicroSerivce)들에 대한 정보 등이 있다

## Eureka client

### 개발 환경

- SpringBoot 2.4.4
- Maven
- Java 11
- SpringCloud 2020.0.2

### 의존성(dependency)

![EurekaClient](/assets/postImages/SpringCloudNetflixEureka/EurekaClient.PNG)

이번에는 서비스 기능을 구현하기 위해서 `Eureka` 외에도 추가적인 의존성을 추가해주었다면
아까는 `Eureka Server`를 추가해주었다면 이번에는 `Eureka Discovery Client`를 추가해주도록 하자

### 설정 파일 수정(application.yml)

```
server:
  port: 8001

spring:
  application:
    name: user-service

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka
```

해당 서비스의 포트와 고유네임, 아까와는 다르게 `register-with-eureka`, `fetch-registry`를 true로 설정한다
추가로 `service-url.defaultZone`를 추가하여 `eureka server`의 url를 작성해주면 된다

### Eureka 클라이언트 실행

``` java
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }

}
```

아까와는 다르게 `@EnableDiscoveryClient` 어노테이션을 추가시켜서 어플리케이션을 실행시켜준다

**이때 유의해야할 점은 아까 작성한 Eureka Server도 마찬가지로 미리 실행시켜주어야 한다**

![eurekaServerRegister](/assets/postImages/SpringCloudNetflixEureka/eurekaServerRegister.PNG)

![eurekaServerRegister2](/assets/postImages/SpringCloudNetflixEureka/eurekaServerRegister2.PNG)

위 사진과 같이 해당 서비스가 서버에 등록되었음을 확인할 수 있다

### 서비스 여러 개 등록 - 1

서비스를 하나만 키는 것이 여러 개를 켜보자 이때 yml 파일에 들어가서 포트를 하드코딩하는 것이 아니라 어플리케이션 실행시 `-Dserver.port=8002`와 같은 옵션을 주어 동적으로 변경하는 것이 바람직하다

``` bash
mvn clean // 이전 빌드 내용 비우기
mvn compile package // 프로젝트 빌드
```

/taget 폴더 안에 따로 설정하지않았다면 `<프로젝트명>-0.0.1-SNAPSOT.jar` 파일이 생성이 된것을 확인할 수 있을 것이다

``` bash
bash창 세개를 열어
java -jar <생성된 jar파일명> --server.port=8001
java -jar <생성된 jar파일명> --server.port=8002
java -jar <생성된 jar파일명> --server.port=8003
```

![eurekaServerRegister4](/assets/postImages/SpringCloudNetflixEureka/eurekaServerRegister4.PNG)

위 사진처럼 세 개의 서비스가 등록된 것을 확인할 수 있다

### 서비스 여러 개 등록 - 2

위와 같이 하게 되면 매번 옵션을 주어 포트를 임의로 지정해줘야하는 것은 불편한 작업이 아닐 수가 없다

``` yml
server:
  port: 0
```

0으로 지정하게되면 랜덤한 포트가 지정이 된다

![eurekaServerRegister5](/assets/postImages/SpringCloudNetflixEureka/eurekaServerRegister5.PNG)

0포트라고 적혀있지만 마우스를 올려보면 왼쪽 하단에 진짜 포트가 적혀있는 것을 확인할 수 있다

하지만 이렇게 하면 서비스를 두개를 키더라도 호스트명과 포트는 0으로 동일하기 때문에 하나로 인식하여 제대로 확인을 할 수가 없다

``` yml
eureka:
  instance:
    instance-id: ${spring.cloud.client.hostname}:${spring.application.instance_id:${random.value}}
```

이를 해결하기 위해서 `instance-id`를 다음과 같이 지정해주게된다

![eurekaServerRegister6](/assets/postImages/SpringCloudNetflixEureka/eurekaServerRegister6.PNG)

두 개 이상의 같은 서비스를 키더라도 구분되어 확인할 수 있을 것이다

긴 글 읽어주셔서 감사합니다.
