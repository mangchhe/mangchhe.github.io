---
title: "[Spring Boot] Filter 구현"
decription: Filter가 간단하게 무엇인지, 구현 방법과 body를 얻어올 때 주의해야 할 점에 대해서 알아보려고 한다.
categories:
 - SpringBoot
tags:
 - SpringBoot
 - Filter
---

> Filter가 간단하게 무엇인지, 구현 방법과 body를 얻어올 때 주의해야 할 점에 대해서 알아보려고 한다.

# Filter란?

<hr>

사용자(Client)와 DispatchServlet 사이에 위치하며 오고 가는 요청과 응답에 대해서 검사하거나 변경할 수 있는 것이다. 그렇기 때문에 전역으로 처리하기 위한 로깅 처리, 인코딩 변환, XSS 방어, 인증 처리에 이용될 수 있다.

![FilterTest0](/assets/postImages/SpringBootFilter/FilterTest0.jpeg)

# 필터 구현

<hr>

servlet 패키지의 Filter interface를 구현하여 필터를 구현할 수 있다.

``` java
@Slf4j
public class FirstFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("FirstFilter start");
        chain.doFilter(request, response);
        log.info("FirstFilter end");
    }

    @Override
    public void destroy() {
        Filter.super.destroy();
    }
}
```

- init : 필터를 생성할 때 실행
- doFilter : 요청/응답 시에 실행
- destory : 필터가 소멸할 때 실행

# 필터 등록

<hr>

설정 파일을 만들어 빈으로 등록한다.

``` java
@Configuration
public class CustomFilterRegister {

    @Bean
    public FilterRegistrationBean firstFilter() {
        FilterRegistrationBean firstFilter = new FilterRegistrationBean(new FirstFilter());
        ArrayList<String> urls = new ArrayList<>(); urls.add("/");
        firstFilter.setUrlPatterns(urls);
        firstFilter.setOrder(0);
        return firstFilter;
    }
}
```

- setUrlPatterns : 필터를 적용할 url을 Collection<String> 타입을 파라미터로 넘겨주면 된다. 없다면 /* 로 간주한다.
- setOrder : 필터의 우선순위를 결정하며 숫자가 낮은 순서대로 높은 우선순위를 가지며 설정하지 않으면 등록된 순서대로 실행된다.

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
}
```

## 결과

![FilterTest1](/assets/postImages/SpringBootFilter/FilterTest1.png)

# 심화

<hr>

컨트롤러, 필터를 두 개씩 더 추가해서 테스트해보자

## Filter

``` java
@Slf4j
public class SecondFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("SecondFilter start");
        chain.doFilter(request, response);
        log.info("SecondFilter end");
    }
}

@Slf4j
public class ThirdFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("ThirdFilter start");
        chain.doFilter(req, response);
        log.info("ThirdFilter end");
    }
}

```

## Bean Regist

``` java
@Bean
public FilterRegistrationBean secondFilter() {
    FilterRegistrationBean secondFilter = new FilterRegistrationBean(new SecondFilter());
    ArrayList<String> urls = new ArrayList<>(); urls.add("/"); urls.add("/second-filter");
    secondFilter.setUrlPatterns(urls);
    secondFilter.setOrder(1);
    return secondFilter;
}

@Bean
public FilterRegistrationBean thirdFilter() {
    FilterRegistrationBean thirdFilter = new FilterRegistrationBean(new ThirdFilter());
    ArrayList<String> urls = new ArrayList<>(); urls.add("/"); urls.add("/third-filter");
    thirdFilter.setUrlPatterns(urls);
    thirdFilter.setOrder(2);
    return thirdFilter;
}
```

## Controller

``` java
@GetMapping("/second-filter")
public String hello2() {
    log.info("/second-filter 진입");
    return "Hello";
}

@GetMapping("/third-filter")
public String hello3() {
    log.info("/third-filter 진입");
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

        mockMvc.perform(MockMvcRequestBuilders.get("/second-filter"))
                .andExpect(MockMvcResultMatchers.status().isOk());

        mockMvc.perform(MockMvcRequestBuilders.get("/third-filter"))
                .andExpect(MockMvcResultMatchers.status().isOk());
    }
}
```

## 표 정리

|-|first|second|third|
|:-:|:-:|:-:|:-:|
|우선순위|0|1|2|
|url patterns|/|/, /second-filter|/, /third-filter|

## 결과

![FilterTest2](/assets/postImages/SpringBootFilter/FilterTest2.png)

/ 진입부터 보게 되면 세 개의 필터 모두 해당 url을 가지고 있기 때문에 전부 실행되는데 이때 우선순위가 first > second > third 이기 때문에 순서대로 실행되는 것을 알 수 있다.

/second-filter 진입 시에는 두 번째 필터만, /third-filter 진입 시에는 세 번째 필터만 진입하는 것을 알 수 있다.

# 주의할 점

<hr>

만약 사용자의 요청 body를 읽으려고 할 때 아래와 같이 소스 코드를 짜게 될 것이다.

``` java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    HttpServletRequest req = (HttpServletRequest) request;

    BufferedReader reader = req.getReader();
    String line = "";
    int lineCnt = 1;
    while ((line = reader.readLine()) != null) {
        log.info(lineCnt + " " + line);
        lineCnt += 1;
    }
    chain.doFilter(req, response);
}
```

![Error](/assets/postImages/SpringBootFilter/Error.png)

다음과 같은 오류를 만나게 되는데 `getReader()` 또는 `getInputStream()`을 이미 사용했다는 에러가 뜨게 된다. 이유는 HttpServletRequset의 위 두 함수는 한 번만 읽을 수 있게 구현이 되어있기 때문에 Spring Context에서 Json 데이터로 Convert할 때 에러가 발생하게 된다.

이 문제를 해결하기 위해서 Wrapper 클래스를 이용해 InputStream을 읽어서 내용을 저장해놓고 계속해서 쓰는 방식을 이용해야 한다.

## 해결 코드

ContentCachingRequestWrapper의 래퍼 클래스를 이용하여 값을 캐시에 저장하여 사용하여 문제를 해결할 수 있다.

``` java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    HttpServletRequest req = (HttpServletRequest) request;
    HttpServletResponse res = (HttpServletResponse) response;

    ContentCachingRequestWrapper requestWrapper = new ContentCachingRequestWrapper(req);
    System.out.println(new String(requestWrapper.getContentAsByteArray()));
    ContentCachingResponseWrapper responseWrapper = new ContentCachingResponseWrapper(res);

    chain.doFilter(requestWrapper, responseWrapper);
    responseWrapper.copyBodyToResponse();
}
```


# Reference

<hr>

- [https://stackoverflow.com/questions/65790323/how-to-get-request-body-params-in-spring-filter](https://stackoverflow.com/questions/65790323/how-to-get-request-body-params-in-spring-filter)
- [https://jronin.tistory.com/124](https://jronin.tistory.com/124)