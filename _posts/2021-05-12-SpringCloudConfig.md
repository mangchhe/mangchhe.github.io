---
layout: post
title: "Implement Spring Cloud Config"
categories: springcloud
tags: springcloud
---

- 분산 시스템에서 설정 파일을 외부로 분리될 수 있게 지원한다.
- 각 서비스에서 분리 된 설정 파일들은 한 곳에서 관리할 수 있다.
- 배포 파이프 라인을 통해 DEV(개발) - UAT(테스트) - PROD(배포) 와 같이 각 환경에 맞는 구성 정보를 사용할 수 있다.
- 설정 파일의 저장소는 Git을 기본으로 사용한다.

# Spring Colud Config를 사용하는 이유?

![netflixArchitectureDiagram](/assets/postImages/SpringCloudConfig/netflixArchitectureDiagram.PNG)

위 사진은 Netflix의 마이크로서비스들의 관계를 나타낸 것이다. 한눈에 봐도 복잡해보이는 이러한 어플리케이션이 구성되기 위해서는 수십~백가지의 마이크로서비스들이 모여 통신한다. 각각의 마이크로서비스들은 설정 파일들을 개발 환경(개발 - 테스트 - 배포)에 따라 하나 이상의 설정 파일들이 있을 것이다.

- 수많은 마이크로서비스들의 설정 파일들을 각각의 서비스에서 따로 관리한다고하면 공통적으로 사용하는 설정의 경우에 해당하는 모든 서비스를 찾아가 변경을 해줘야하기 때문에 유지 보수에 큰 어려움을 겪을 수 밖에 없다
-> **모든 설정 파일들은 프로젝트와 분리하여 관리할 수 있게 되어 이전과는 쉽게 관리할 수 있게 지원한다.**
- 설정 파일이 서비스 내부에 종속되어 있으면 설정 파일이 변경이 일어났을 때마다 적용시켜주기 위해서 해당 서비스를 전부 빌드 후에 재구동시켜야 하기 때문에 큰 부담이 될 수 밖에 없다
-> **재구동을 할 필요 없이 서비스 도중에 설정을 변경 할 수 있게 지원한다.**

# 실습 전 사전지식

## Spring 설정 파일 우선순위

1. bootstrap.yml
  - application.yml 보다 먼저 로드 되며 필요한 정보들을 먼저 가져오기 위해 spring cloud config에 대한 정보를 작성한다.
2. applcation.yml
  - 기존에 적은 포트 정보나 어플리케이션 이름을 작성한다.

## Spring Cloud Config 설정 파일 우선순위

1. application-name-<profile>.yml
  - 해당 서비스들의 환경에 따라 엔드포인트들을 지정할 수 있고(dev, test, prod 등) 그에 맞는 설정 내용을 담고 있다.
2. application-name.yml
  - 해당 서비스가 공통적으로 가질 수 있는 설정 내용을 담고 있다.
3. application.yml
  - 모든 서비스들이 공통적으로 가질 수 있는 설정 내용을 담고 있다.

기본적으로 서비스에는 application.yml이 들어가 있는데 이 설정 파일들을 한 곳에서 관리하기 위해서는 식별 할 수 있어야 하기 때문에 설정 파일에 각 서비스 이름을 부여해줘야 할 것이다. 이때 작명에 따라서 우선순위가 나눠질 수 있다.

## Configuration 저장 세 가지 방법

- Local Git Repository를 이용하는 방법
  - Remote 저장소에 보내지 않고도 로컬 환경에서 커밋만을 통해 configuration을 이용할 수 있다.
- Remote Git Repository를 이용하는 방법
  - Remote 저장소에 Configuration 정보를 보냄으로써 다른 쪽에서도 해당 정보를 이용할 수 있다.
- Native File Repository를 이용하는 방법
  - 로컬에 있는 파일 시스템을 이용하여 Configuration를 이용할 수 있다.

## 변경 된 Configuration 정보를 최신화하는 세 가지 방법

- 서비스 재기동
- Actuator refresh
- Spring Cloud Bus 사용

# 실습 1 - Local Git Repository

## 사전작업

### git repository 생성

``` bash
# 설정 파일을 저장할 디렉토리로 이동
git init
# 설정 파일 변경 시 반복
git add <추가/변경 된 파일명>
git commit -m "commit message"
```

### ecommerce.yml 생성

``` yaml
token:
  expiration_time: 864000000
  secret: user_token_default
```

Local Git Repository를 생성한 후 그곳에 설정 파일을 생성하고 해당 서비스에 필요한 설정 내용들을 작성한다.

## ConfigService 프로젝트 생성

### 의존성 추가

``` java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

### main 수정

``` java
@SpringBootApplication
@EnableConfigServer
public class ConfigServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServiceApplication.class, args);
    }

}
```

### applcation.yml 수정

``` yml
server:
  port: 8888

spring:
  application:
    name: config-service
  cloud:
    config:
      server:
        git:
          uri: file:///{Local Repository PATH}
```

### ConfigService 구동

http://localhost:8888/{yml명}/{profile명} 에 요청하게 되면 아까 전에 생성하였던 ecommerce.yml이 조회되는 것을 확인할 수 있다.

![configLocalReposResult](/assets/postImages/SpringCloudConfig/configLocalReposResult.PNG)

## MicroService - ConfigService 연동

### MicroService

#### 의존성 추가

``` java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

#### bootstrap.yml 추가

``` yml
spring:
  cloud:
    config:
      uri: http://127.0.0.1:8888
      name: ecommerce
```

config service의 uri와 가져올 설정 파일의 이름을 작성하고 application.yml과 동일 PATH 상에 파일을 생성한다.

#### Controller 수정

``` java
@RestController
@RequiredArgsConstructor
@RequestMapping("/")
public class UserController {
    private final Environment env;

    @GetMapping("health_check")
    public String status(){
        return String.format("It's Working in User Service"
                + ", local port : " + env.getProperty("local.server.port")
                + ", port : " + env.getProperty("server.port")
                + ", secret : " + env.getProperty("token.secret")
                + ", expiration_time : " + env.getProperty("token.expiration_time"));
    }
}
```

연동할 서비스의 컨트롤러에 내용이 제대로 적용이 되었는지 기존 서비스 applcation.yml 설정되어 있는 서버 포트와 config service에서 가져온 port, secret, expiration_time 정보를 확인할 수 있는 엔드포인트를 작성한다.

### 서비스 구동

![configLocalReposResult2](/assets/postImages/SpringCloudConfig/configLocalReposResult2.PNG)

http://127.0.0.1:<포트번호>/health_check 에 요청을 보내게 되면 applcation.yml에 있는 port 정보와 config service에 있는 ecommerce.yml의 secret, expiration_time 정보와 port는 따로 정보를 준것이 없기 때문에 0으로 나타나있는 것을 확인할 수 있다.

## Profiles 설정

### 저장소에 yml 추가

``` yml
# ecommerce-dev.yml
token:
  expiration_time: 86400000
  secret: user_token_dev#1

# ecommerce-prod.yml
token:
  expiration_time: 86400000
  secret: user_token_prod#1
```

> #### MicroService bootstrap.yml 수정

``` yml
# 개발 환경
spring:
  cloud:
    config:
      uri: http://127.0.0.1:8888
      name: ecommerce
  profiles:
    active: dev

# 배포 환경
spring:
  cloud:
    config:
      uri: http://127.0.0.1:8888
      name: ecommerce
  profiles:
    active: prod
```

`spring.cloud.profiles.active` 에 적용하고 싶은 설정 파일 하이폰 뒤에 주었던 profile 명을 기입한다.

### 테스트

![configLocalReposResult3](/assets/postImages/SpringCloudConfig/configLocalReposResult3.PNG)
![configLocalReposResult4](/assets/postImages/SpringCloudConfig/configLocalReposResult4.PNG)

http://127.0.0.1:<포트번호>/health_check 에 요청을 보내게 되면 설정한 profile 환경에 맞게 설정이 되는 것을 확인할 수 있다.

# 실습 2 - Remote Git Repository

[깃허브](https://github.com/)로 이동하여 설정 파일을 저장할 저장소를 하나 생성한다.

## Git 연동

``` bash
# 설정 파일을 저장할 디렉토리로 이동
# 원격 저장소 연동 방법 1
git init
git remote add origin <Git Repository URL>
git add .
git commit -m "commit message"
git push --set-upstream origin master
# 원격 저장소 연동 방법 2
git clone <Git Repository URL>
git add .
git commit -m "commit message"
git push
```

## ConfigService application.yml 수정

``` yml
spring:
  cloud:
    config:
      server:
        git:
          # uri: {Local Repository PATH}
          uri: https://github.com/mangchhe/WEB_Cloud_Tutorial_Config.git
#         username: [username]
#         password: [password]
```

이전에 했던 Local Repository 부분을 주석처리하고 uri 부분에 생성했던 깃허브 주소를 입력한다. username, password 같은 경우에는 공개여부가 public일 경우 설정해줄 필요가 없다.

# 실습 3 - Native File Repository

로컬에 설정 파일을 담은 디렉토리를 하나 생성하고 각종 설정 파일들을 담는다.

## ConfigService application.yml 수정

``` yml
spring:
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          # uri: {Local Repository PATH}
          # uri: {Remote Repository PATH}
          search-locations: file:///<native-repository-url>
```

## 결과

![configNativeReposResult](/assets/postImages/SpringCloudConfig/configNativeReposResult.PNG)

현재 로컬 디렉토리에는 다음과 같은 5개의 설정파일들이 존재하고 config service를 구동시켜 설정 파일이 정상 등록이 되었는지 확인해보자

http://localhost:8888/user-service/default

![configNativeReposResult2](/assets/postImages/SpringCloudConfig/configNativeReposResult2.PNG)

http://localhost:8888/user-service/dev

![configNativeReposResult3](/assets/postImages/SpringCloudConfig/configNativeReposResult3.PNG)

http://localhost:8888/ecommerce/default

![configNativeReposResult4](/assets/postImages/SpringCloudConfig/configNativeReposResult4.PNG)

로컬에 있는 각 설정 파일들이 정상적으로 등록되어있는 것을 확인할 수 있다.
