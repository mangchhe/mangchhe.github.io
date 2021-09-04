---
title: 【Spring Cloud】 Gateway로 Filter, LoadBalancer 구현
decription: Spring Cloud Gateway를 구현하는 방법부터 여러 종류의 필터들과 로드밸런싱 적용에 대해서 알아보자
categories:
 - SpringCloud
tags:
 - Spring Cloud
 - MSA
 - Spring Cloud Gateway
 - API Gateway
 - Filter
 - Load Balancer
---

> Spring Cloud Gateway를 구현하는 방법부터 여러 종류의 필터들과 로드밸런싱 적용에 대해서 알아보자

> ## Spring Cloud Gateway란?

- API Gateway → [클릭](https://mangchhe.github.io/springcloud/2021/04/08/ApiGatewayConcept/)
- MSA에서 여러 클라이언트 요청을 적절한 서비스로 보내기 위한 Proxy 역할을 한다

> ## Spring Cloud Gateway 구현

- 앞서 [포스트](https://mangchhe.github.io/springcloud/2021/04/08/NetflixZuul/)에서 만든 first-service, second-service 두 마이크로서비스와 함께 Zuul 대신해서 Spring Cloud Gateway를 구현해보자
- **개발 환경** : SpringBoot 2.4.4, Maven, Java 11
- **소스 코드** : [링크](https://github.com/mangchhe/WEB_Cloud_Tutorial)

> ### API Gateway

> #### 의존성(dependency)

``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
    <version>3.0.2</version>
</dependency>
```

> #### applications.yml

``` yml
server:
  port: 8080
spring:
  application:
    name: apigateway-service
  cloud:
    gateway:
      routes:
        - id: first-service
          uri: http://localhost:8081/
          predicates:
            - Path=/first-service/**
        - id: second-service
          uri: http://localhost:8082/
          predicates:
            - Path=/second-service/**
```

#### cloud.gateway.routes

- id : 식별 할 수 있는 값
- uri : 목적지 주소
- predicates : 목적지 주소로 이동하기 위한 조건

`/first-service` 해당 경로로 들어오는 모든 요청은 목적지 `http://localhost:8081/`로 간다는 것을 의미

> #### 결과

![SpringCloudGatewayResult1](/assets/postImages/SpringCloudGateway/SpringCloudGatewayResult1.PNG)

![SpringCloudGatewayResult2](/assets/postImages/SpringCloudGateway/SpringCloudGatewayResult2.PNG)

API Gateway와 서비스를 실행한 후에 요청하면 목적지로 잘 이동한 것을 확인할 수 있다

> ## Filter 적용

![SpringCloudGatewayFilterConcept](/assets/postImages/SpringCloudGateway/SpringCloudGatewayFilterConcept.PNG)

클라이언트에서 Spring Cloud Gateway로 요청을 하게 될 경우 Gateway Handler에서 요청의 경로가 일치하다고 생각하면 이 핸들러는 요청에 필요항 특정한 필터 체인을 통해 요청을 실행한다
GlobalFilter, CustomFilter를 거치며 인증&권한, 로깅과 같은 역할을 할 수 있다

### 필터 적용 방법
- application.yml 수정
- RouteLocator 객체 Bean 등록

필터를 적용하는 두 가지 방법 각각 요청과 응답 헤더에 키(<서비스번호>-request/response), 값(<서비스번호>-request/response-header2)을 추가 하는 것을 알아보자

> ### application.yml 수정하는 방법

``` yml
server:
  port: 8080
spring:
  application:
    name: apigateway-service
  cloud:
    gateway:
      routes:
        - id: first-service
          uri: http://localhost:8081/
          predicates:
            - Path=/first-service/**
          filters:
            - AddRequestHeader=first-request, first-request-header2
            - AddResponseHeader=first-response, first-response-header2
        - id: second-service
          uri: http://localhost:8082/
          predicates:
            - Path=/second-service/**
          filters:
            - AddRequestHeader=second-request, second-request-header2
            - AddResponseHeader=second-response, second-response-header2
```

#### cloud.gateway.routes

`filters`를 추가하고 - AddRequestHeader, addResponseHeader에 Key, Value 값을 넣어주게 되면 요청할 때 <서비스번호>-request 키로 <서비스번호>-request-header2 라는 값이 요청 헤더에 들어가게 되고 그에 반해 응답할 때는 <서비스번호>-response 키로 <서비스번호>-response-header2 라는 값이 응답 헤더에 들어가게 될 것이다

> ### RouteLocator 객체 등록하는 방법

``` java
@Configuration
public class FilterConfig{
    @Bean
    public RouteLocator gatewayRoutes(RouteLocatorBuilder builder){
        return builder.routes()
                .route(r -> r.path("/first-service/**")
                            .filters(f -> f.addRequestHeader("first-request", "first-request-header")
                            .addResponseHeader("first-response", "first-response-header"))
                            .uri("http://localhost:8081")
                )
                .route(r -> r.path("/second-service/**")
                            .filters(f -> f.addRequestHeader("second-request", "second-request-header")
                            .addResponseHeader("second-response", "second-response-header"))
                            .uri("http://localhost:8082")
                )
                .build();
    }
}
```

`RouteLocatorBuilder`을 이용하여 요청 또는 응답에 대한 predicates(조건)와 filters를 구현하여 Bean으로 다음과 같이 구현하게 되면 application.yml에 적용한 것과 같은 동작을 할 수 있게 된다

> ### FirstService, SecondService

``` java
@Slf4j
public class FirstServiceController {

    ...

    @GetMapping("/message")
    public String message(@RequestHeader("first-request") String header){
      log.info(header);
      return "Hello World in First Service.";
    }
}
```

FirstService, SecondService에 요청헤더에 필터에 적용시킨 값이 들어왔는지 확인하기 위해 /message 경로와 @RequestHeader("first-request")를 추가시켜 주어 로그 기록을 구현하였다

> ### 결과

![SpringCloudGatewayFilterResult1](/assets/postImages/SpringCloudGateway/SpringCloudGatewayFilterResult1.PNG)

요청을 했을 때 first-request-header가 추가된 것을 확인할 수 있다

![SpringCloudGatewayFilterResult2](/assets/postImages/SpringCloudGateway/SpringCloudGatewayFilterResult2.PNG)

응답을 받았을 때 first-response-header가 추가 된 것을 확인할 수 있다

> ## CustomFilter 적용

Filter를 이용해서 요청이 들어올 경우 요청된 ID, URI와 응답 시 응답 코드를 로그를 기록하는 전/후처리 필터를 구현해보자

> ### CustomFilter Class

``` java
@Component
@Slf4j
public class CustomFilter extends AbstractGatewayFilterFactory<CustomFilter.Config> {
    public CustomFilter() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        // Custom Pre Filter
        return ((exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            ServerHttpResponse response = exchange.getResponse();

            log.info("Cutsom Pre filter : request id => {}", request.getId());
            log.info("Cutsom Pre filter : request uri => {}", request.getURI());

            // Custom Post Filter
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                 log.info("Custom Post filter : response code -> {}", response.getStatusCode());
            }));
        });
    }

    public static class Config {
        // Put the configuration properties
    }
}
```

AbstractGatewayFilterFactory를 상속 받아 구현하고 apply 메소드에 필터가 수행할 로직에 대해서 구현한다 Spring Cloud APIGateway는 동기가 아닌 비동기 방식이기 때문에 HttpServletRequest와 같은 서블릿이 아닌 SeverHttpReuqest/Response를 이용한다 또한 반환 값도 마찬가지로 Mono, Flux를 이용하여 비동기 방식으로 구현하게 된다

> ### application.yml

``` yml
- id: first-service
  uri: http://localhost:8081/
  predicates:
    - Path=/first-service/**
  filters:
    - CustomFilter
- id: second-service
  uri: http://localhost:8082/
  predicates:
    - Path=/second-service/**
  filters:
    - CustomFilter
```

위에서 구현된 CustomFilter를 filters 밑에 작성하게 되면 해당 서비스에 요청이 들어갈 경우 필터가 적용이 된다

> ### 결과

![SpringCloudGatewayCustomFilterResult1](/assets/postImages/SpringCloudGateway/SpringCloudGatewayCustomFilterResult1.PNG)

요청이 들어왔을 때 요청 id, uri가 로그에 기록되고 응답시 응답코드가 기록된 것을 확인할 수 있다

> ## GlobalFilter 적용

각각의 필터를 필요로 하는 서비스에 개별적으로 넣어주는 CustomFilter와 다르게 전체적으로 적용되어야 되는 인증/권한과 같은 필터와 같은 경우에는 GlobalFilter로 구현을 하게 될 것이다 GlobalFilter를 구현하여 로그를 찍어보자

> ### Global Filter Class

``` java
@Component
@Slf4j
public class GlobalFilter extends AbstractGatewayFilterFactory<GlobalFilter.Config> {
    public GlobalFilter() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        return ((exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            ServerHttpResponse response = exchange.getResponse();

            log.info("Global Filter baseMessage : {}", config.getBaseMessage());

            if (config.isPreLogger()){
                log.info("Global Filter Start : request id -> {}", request.getId());
            }

            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                if (config.isPostLogger()){
                    log.info("Global Filter End : response code -> {}", response.getStatusCode());
                }
                log.info("Custom Post");
            }));
        });
    }

    @Getter @Setter
    public static class Config {
        String baseMessage;
        private boolean preLogger;
        private boolean postLogger;
    }
}
```

Filter 구현은 CustomFilter 구현과 동일하다 AbstractGatewayFilterFactory을 상속받아 apply 메소드에 필요한 로직을 구현한다 내용이 추가 된것을 Config에 값을 추가하였고 필터가 공통적으로 사용할 변수들에 대해서 정의하였다


> ### application.yml

``` yml
cloud:
  gateway:
    routes:
      - id: first-service
        uri: http://localhost:8081/
        predicates:
          - Path=/first-service/**
        filters:
          - CustomFilter

      ...

    default-filters:
      - name: GlobalFilter
        args:
          baseMessage: Spring Cloud Gateway Global Filter
          preLogger: true
          postLogger: true
```

서비스에 개별적으로 추가시켜주던 CustomFilter와는 다르게 default-filters를 추가하여 GlobalFilter를 적용한다

> ### 결과

![SpringCloudGatewayGlobalFilterResult1](/assets/postImages/SpringCloudGateway/SpringCloudGatewayGlobalFilterResult1.PNG)

**GlobalFilter pre** - CustomFilter pre/post - **GlobalFilter post** 순으로 적용된 것을 확인할 수 있다

> ## LoggingFilter 적용

LogginFilter라는 CustomFilter를 하나 더 구현하려고 한다 이전 CustomFilter와 다르게 Config에 필터가 사용할 변수들을 추가하고 적용하는 방법과 apply 메소드 처리 로직에 대해서 순서까지 등록해보자

> ### LoggingFilter Class

``` java
@Component
@Slf4j
public class LoggingFilter extends AbstractGatewayFilterFactory<LoggingFilter.Config> {
    public LoggingFilter() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {

        GatewayFilter filter = new OrderedGatewayFilter((exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            ServerHttpResponse response = exchange.getResponse();

            log.info("Logging Filter baseMessage : {}", config.getBaseMessage());

            if (config.isPreLogger()){
                log.info("Logging PRE Filter : request id -> {}", request.getId());
            }

            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                if (config.isPostLogger()){
                    log.info("Logging POST Filter : response code -> {}", response.getStatusCode());
                }
            }));

        }, Ordered.LOWEST_PRECEDENCE);

        return filter;
    }

    @Getter @Setter
    public static class Config {
        private String baseMessage;
        private boolean preLogger;
        private boolean postLogger;
    }
}
```

하고자하는 내용은 이전 CustomFilter와 동일하게 전/후처리시에 내용을 기록한다

((exchange, chain) -> {}) 내용을 그대로 반환했던 반면 이제는 GatewayFilter filter를 만들어서 반환하게 된다 이때 OrderedGatewayFilter를 이용하게 되는데 내용을 들여다보자

``` java
public class OrderedGatewayFilter implements GatewayFilter, Ordered {
    private final GatewayFilter delegate;
    private final int order;

    public OrderedGatewayFilter(GatewayFilter delegate, int order) {
        this.delegate = delegate;
        this.order = order;
    }

    public GatewayFilter getDelegate() {
        return this.delegate;
    }

    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        return this.delegate.filter(exchange, chain);
```

OrderedGatewayFilter는 GatewayFilter와 Ordered를 상속받고 있으며 첫번째 파라미터로 Mono<Void> filter를 이용하여 GatewayFilter를 구현하고 두번째 파라미터로 Ordered 순서를 입력받게 받아 구현하게 된다

```
public interface Ordered {
    int HIGHEST_PRECEDENCE = -2147483648;
    int LOWEST_PRECEDENCE = 2147483647;

    int getOrder();
}
```
Ordered는 다음과 같이 구현되어 있고 숫자가 낮을수록 높은 우선순위를 가진다
현재는 LOWEST_PRECEDENCE를 가지고 있어 가장 마지막에 실행될 것이다

> ### application.yml

``` yml
- id: second-service
  uri: http://localhost:8082/
  predicates:
    - Path=/second-service/
  filters:
    - CustomFilter
    - name: LoggingFilter
      args:
        baseMessage: Spring Cloud Gateway Logging Filter
        preLogger: true
        postLogger: true
```

CustomFilter에 Config 정보를 설정하기 위해서는 이전에 GlobalFilter에서 헀던 것과 같이 args.baseMessage/preLogger 등 값을 입력해주면 된다 이때 정보를 넣어주기 위해서는 이전 CustomFilter와 같이 필터명만 적는 것이 아닌 앞에 -name 속성을 붙여서 작성해주어야 한다

> ### 결과

![SpringCloudGatewayLoggingFilterResult1](/assets/postImages/SpringCloudGateway/SpringCloudGatewayLoggingFilterResult1.PNG)

GlobalFilter pre - CustomFilter pre - **LoggingFilter pre/post** - CustomFilter post - GlobalFilter post와 같이 LogginFilter가 마지막에 실행되는 것을 확인할 수 있다

> ## 로드 밸런서(Load Balancer) 적용

같은 로직을 수행하는 서비스 로직이 두개가 있을 경우에 로드밸런싱을 구현하여 트래픽을 분산시켜보자

> ### Eureka Service

이전 [포스트](https://mangchhe.github.io/springcloud/2021/04/06/SpringCloudNetflixEureka/)에서 구현했던 내용 그대로 사용

> ### FirstService, SecondService

> #### application.yml

``` java
eureka:
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    instance-id: ${spring.application.name}:${spring.application.instance_id:${random.value}}
```

Eureka(Service Discovery)에 first-service, second-service를 등록시켜주기 위해 다음과 같이 내용을 추가해준다

instance-id를 설정해주는 이유는 포트를 0으로 설정하게 되면 랜덤 포트가 설정되는데 실제로 Eureka 대쉬보드에는 0으로 표기되어 서비스가 목록에는 하나밖에 표기되지 않아 따로 설정을 해주어야 하는 것이다

> #### Controller

``` java
@GetMapping("/check")
public String check(HttpServletRequest request){

    log.info("Server port={}", request.getServerPort());

    return String.format("First Service Port %s", env.getProperty("local.server.port"));
}
```

로드밸런싱 되어 들어오는 요청이 어떤 서비스에 들어오는지 알기 위해서 해당 서비스에 들어오게 되면 로그에 포트가 기록되도록 Controller를 수정한다

> ### Spring Cloud Gateway

``` yml
spring:
  cloud:
    gateway:
      routes:
        - id: first-service
          // uri: http://localhost:8081/
          uri: lb://FIRST-SERVICE
        - id: second-service
          // uri: http://localhost:8082/
          uri: lb://SECOND-SERVICE

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    instance-id: ${spring.application.name}:${spring.application.instance_id:${random.value}}
```

기존에 설정되어 있는 uri를 앞에 lb라는 접두사, 뒤에는 어플리케이션 서비스 이름으로 수정해주고 Eureka Service에 등록될 수 있게 설정해준다

> ## 결과

![SpringCloudGatewayRoad1](/assets/postImages/SpringCloudGateway/SpringCloudGatewayRoad1.PNG)

Eureka Service와 Spring Cloud Gateway를 실행한 후 같은 내용의 서비스를 두 개 실행한 것을 확인할 수 있다

![SpringCloudGatewayRoad2](/assets/postImages/SpringCloudGateway/SpringCloudGatewayRoad2.PNG)
![SpringCloudGatewayRoad3](/assets/postImages/SpringCloudGateway/SpringCloudGatewayRoad3.PNG)


요청을 하게 되면 서로 다른 포트에 요청이 분산되는 것을 로그를 보고 확인할 수 있다

> ## 참고

[Spring Cloud Gateway Filter 사진](https://www.baeldung.com/spring-cloud-custom-gateway-filters)
