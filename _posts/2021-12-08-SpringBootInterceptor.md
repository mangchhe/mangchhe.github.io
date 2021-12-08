---
title: "[Spring Boot] Interceptor 구현"
decription: Spring Boot 인터셉터 구현
categories:
 - SpringBoot
tags:
 - SpringBoot
 - Interceptor
---

> Spring Boot 인터셉터 구현

# 인터셉터(Interceptor)란?

<hr>

DispatcherServlet과 컨트롤러 사이에 위치하며 **컨트롤러에 요청이 들어가기 전에 가로채 특정 작업**을 하거나 **컨트롤러에서 요청에 대한 작업이 끝나고 응답할 때 추가적인 작업을 하는 것**을 말한다. 그렇기에 로깅 또는 인증/인가와 같은 공통 작업을 처리하기위해 많이 사용한다.

![Interceptor0](/assets/postImages/SpringBootInterceptor/Interceptor0.png)

# 실습 환경

<hr>

- Spring Boot 2.6.0
- Java 11

깃허브 링크 : [클릭](https://github.com/mangchhe/Spring_Tutorial/tree/main/Interceptor/Java)

# 인터셉터 구현

<hr>

servlet 패키지의 HandlerInterceptor interface를 구현하여 인터셉터를 구현할 수 있다.

``` java
@Slf4j
public class FirstFilter implements Filter {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info("First Interceptor preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("First Interceptor postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    }
}
```

- preHandle : 컨트롤러가 호출되기 전 실행
- postHandle : 컨트롤러 작업이 완료된 후 실행
- afterCompletion : 뷰에서 최종 결과를 렌더링한 후에 실행

# 인터셉터 등록

<hr>

설정 파일을 만들어 빈으로 등록한다.

``` java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new FirstInterceptor())
                .order(1)
                .addPathPatterns("/*")
                .excludePathPatterns("/second-interceptor");
    }
}
```

- order : 인터셉터 실행 우선순위를 결정한다. 낮을 수록 높은 우선순위를 가진다.
- addPathPatterns : 인터셉터를 적용시킬 url 등록
- excludePathPatterns : 제외할 url 등록

# 테스트

<hr>

컨트롤러를 등록하고 테스트 코드를 작성해서 확인해보자

## Controller

``` java
@RestController
@Slf4j
public class HelloController {
    @GetMapping("/")
    public String hello() {
        log.info("/ 진입");
        return "Hello";
    }
}
```

## Test Code

``` java
@SpringBootTest
@AutoConfigureMockMvc
class HelloControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void enter_root() throws Exception {
        //when, then
        mockMvc.perform(MockMvcRequestBuilders.get("/"))
                .andExpect(MockMvcResultMatchers.status().isOk());
}
```

## 결과

![Interceptor1](/assets/postImages/SpringBootInterceptor/Interceptor1.png)

# 심화

<hr>

컨트롤러, 인터셉터를 두 개씩 더 추가해서 테스트해보자

## Interceptor

``` java
@Slf4j
public class SecondInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info("Second Interceptor preHandle");
        return true;
    }
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("Second Interceptor postHandle");
    }
}
@Slf4j
public class ThirdInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info("Third Interceptor preHandle");
        return true;
    }
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("Third Interceptor postHandle");
    }
}
```

## Bean Regist

``` java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new FirstInterceptor())
                ...;
        /* 추가 */
        registry.addInterceptor(new SecondInterceptor())
                .order(0)
                .addPathPatterns("/", "/second-interceptor");
        registry.addInterceptor(new ThirdInterceptor())
                .order(2)
                .addPathPatterns("/**")
                .excludePathPatterns("/second-interceptor");
    }
}
```

## Controller

``` java
@GetMapping("/second-interceptor")
public String hello2() {
    log.info("/second-interceptor 진입");
    return "Hello";
}

@GetMapping("/third/interceptor")
public String hello3() {
    log.info("/third-interceptor 진입");
    return "Hello";
}
```

## Test Code

``` java
@SpringBootTest
@AutoConfigureMockMvc
class HelloControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void enter_root() throws Exception {
        //when, then
        mockMvc.perform(MockMvcRequestBuilders.get("/"))
                .andExpect(MockMvcResultMatchers.status().isOk());

        mockMvc.perform(MockMvcRequestBuilders.get("/second-interceptor"))
                .andExpect(MockMvcResultMatchers.status().isOk());

        mockMvc.perform(MockMvcRequestBuilders.get("/third/interceptor"))
                .andExpect(MockMvcResultMatchers.status().isOk());
    }
}
```

## 표 정리

|-|first|second|third|
|:-:|:-:|:-:|:-:|
|우선순위|1|0|2|
|url patterns|/*|/, /second-interceptor|/**|
|exclude patterns|/second-interceptor|-|/second-interceptor|

## 결과

![Interceptor2](/assets/postImages/SpringBootInterceptor/Interceptor2.png)

- <font color='red'>/ 진입</font>
    - 세 개의 인터셉터를 보면 /*, /, /**를 포함하고 있기 때문에 전부 실행되는데 이때 우선순위가 second > first > third 이기 때문에 순서대로 실행된다.
- <font color='blue'>/second-interceptor 진입</font>
    - first, third 인터셉터는 exclude url에 /second-interceptor가 포함되어 있기 때문에 second 인터셉터만 접근한다.
- <font color='green'>/third/interceptor</font>
    - /** 가지는 third 인터셉터만 접근이 가능하다.

> /*, /** 차이
- /* : / 다음에 오는 모든 문자열을 포함하는 경로를 말함
- /** : / 를 포함하는 모든 문자열과 그 아래 하위 모든 경로도 포함함

# Reference

<hr>

- [https://myhappyman.tistory.com/199](https://myhappyman.tistory.com/199)