---
title: 【Spring Cloud】 Netflix Zuul에 대해서 실습을 통해 알아보자
decription: 서비스들을 찾아가기 위해서 거치게 관문과 같은 역할을 하는 Zuul API Gateway에 대해서 알아보자
categories:
 - SpringCloud
tags:
 - Spring Cloud
 - MSA
 - Netflix Zuul
 - API Gateway
---

> 서비스들을 찾아가기 위해서 거치게 관문과 같은 역할을 하는 Zuul API Gateway에 대해서 알아보자

> ## Zuul이란?

- API Gateway → [클릭](https://mangchhe.github.io/springcloud/2021/04/08/ApiGatewayConcept/)
- MSA에서 여러 클라이언트 요청을 적절한 서비스로 보내기 위한 Proxy 역할을 한다

![zuul_maintenance](/assets/zuul_maintenance.PNG)

netflix-zuul이 maintenance 모드가 되어 더이상 개발 및 패치가 이루어지지 않기 때문에 간단하게 다뤄보려고 한다

![zuul_maintenance2](/assets/zuul_maintenance2.PNG)

위 사진을 보면 Zuul을 대체품으로 Spring Cloud Gateway를 사용하는 것을 추천한다

> ## 실습

![zuul_ex](/assets/zuul_ex.PNG)

두 개의 서비스를 만들고 Zuul을 이용하여 요청이 왔을때 각 서비스에 보내는 방법에 대해서 배우고 필터 등록까지 해보자

> ### 개발 환경

- SpringBoot 2.3.9
  - Zuul은 2.4부터 지원하지 않기때문에 2.4버전 아래 버전을 골라 선택하면된다
- Maven
- Java 11

> ### FirstSerivce&SecondService

> #### 의존성(dependency)

![zuul_service](/assets/zuul_service.PNG)

> #### application.yml

``` yml
server:
  port: 8081

spring:
  application:
    name: first-service

eureka:
  client:
    fetch-registry: false
    register-with-eureka: false
```

``` yml
server:
  port: 8082

spring:
  application:
    name: second-service

eureka:
  client:
    fetch-registry: false
    register-with-eureka: false
```

> #### Controller

``` java
@RestController
@RequestMapping("/")
public class FirstServiceController {
    @GetMapping("/welcome")
    public String welcome(){
        return "Welcome to the First Service";
    }
}
```

``` java
@RestController
@RequestMapping("/")
public class FirstServiceController {
    @GetMapping("/welcome")
    public String welcome(){
        return "Welcome to the Second Service";
    }
}
```

> ### ZuulService

> #### 의존성(dependency)

![zuul_config](/assets/zuul_config.PNG)

> #### application.yml

``` yml
server:
  port: 8080

spring:
  application:
    name: zuul-service

zuul:
  routes:
    first-serivce:
      path: /first-service/**
      url: http://localhost:8081
    second-service:
      path: /second-service/**
      url: http://localhost:8082
```

zuul.routes 밑에 임의에 이름으로 식별가능한 두개의 서비스명을 적고 경로(path)로 요청을 할 경우 어디로(url) 전달할 지를 결정한다

> #### main

``` java
@SpringBootApplication
@EnableZuulProxy
public class ZuulServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZuulServiceApplication.class, args);
    }
}
```

`@EnableZuulProxy`를 적어 ZuulService임을 알려주자

> ### 결과

![zuul_firstservice](/assets/zuul_firstservice.PNG)
![zuul_secondservice](/assets/zuul_secondservice.PNG)

localhost:8080/first-service/wecome -> localhost:8081/first-service/welcome

localhost:8080/second-service/wecome -> localhost:8081/second-service/welcome

localhost:8080으로 요청을 하면 zuul에서 요청에 맞는 서비스를 찾아 정보를 전달하게 되고 처리 후 응답을 받아 클라이언트에게 전달하게 된다

> ## 필터 적용

> ### ZuulService

``` java
@Slf4j
@Component
public class ZuulLoggingFilter extends ZuulFilter {

    @Override
    public Object run() throws ZuulException {
        log.info("************* printing logs :");
        RequestContext currentContext = RequestContext.getCurrentContext();
        HttpServletRequest request = currentContext.getRequest();
        log.info("************* printing logs :" + request.getRequestURI());
        return null;
    }

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }
}
```

ZullFilter를 상속받아 필요한 메소드들을 구현하고 `@Component`를 달아 Bean으로 등록한다
해당 필터는 요청이 들어올 경우 요청한 url에 대해서 로깅을 남긴다
ZuulFilter는 HttpServletRequest를 다루지 않기 때문에 최상위에 있는 RequestContext를 이용하여 해당 객체를 가져와 url을 얻어낸다

`filterType`을 통해 필터가 언제 실행될 것인지를 정할 수 있고 종류에는 pre, route, post, error가 있다
- pre : 요청이 라우팅되기 전에 필터 실행(전처리)
- route : 요청에 추가적인 처리를 실행(추가처리)
- post : 요청이 라우팅된 후에 필터 실행(후처리)
- error : 요청을 처리하는 동안 오류 발생시 필터 실행(오류처리)

`filterOrder`을 통해 등록된 필티 간에 실행 순서를 정할 수 있다

`shouldFilter`을 통해 해당 필터의 사용 여부를 정할 수 있다

`run`을 통해 해당 필터의 동작을 구현한다

> ### 결과

![zuulFilterLogging](/assets/zuulFilterLogging.PNG)

다음과 같이 요청이 들어오면 url에 대한 정보가 기록되는 것을 확인할 수 있다

> ## 참고

[Zuul maintenance](https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now#spring-cloud-netflix-projects-entering-maintenance-mode)
[Zuul Reference](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.1.RELEASE/reference/html/)
