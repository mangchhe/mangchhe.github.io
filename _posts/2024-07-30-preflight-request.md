---
layout: post
title: Preflight Request
categories: web
tags: web
---

CORS 요청을 처리하기 위한 요청하는 방법 중 하나로 Non-Simple Request에 대해서 Preflight 요청을 보내어 실제 원본 요청 전에 허용 여부를 판단한다.

> **Simple Request**
>
> - It is a GET, HEAD, or POST request
> 
> - Only sends automatic User-Agent headers or CORS Safelisted Headers such as Accept, Accept-Language, Content-Language, Content-Type
> 
> - The Content-Type header can only have the values application/x-www-form-urlencoded, multipart/form-data and text/plain
>
> - Does not use a ReadableStream object
>
> - XMLHttpRequest.upload does not have an event listener attached to it

<p align="center">
<img src='https://www.baeldung.com/wp-content/uploads/sites/4/2021/01/Screenshot-2021-01-13-at-23.13.47.png' width="550" height="450">
</p>

Preflight 요청은 OPTIONS 요청으로, 다음과 같은 헤더를 포함한다.

- origin : 요청이 어디서 오는지에 대한 정보
- access-control-request-method : 요청 HTTP Method
- access-control-request-headers : 요청에 포함된 헤더 목록

서버는 다음과 같은 헤더에 값을 보내어 요청에 대해서 허용할지 결정한다.

- access-control-allow-origin : 허용하는 origin 정보
- access-control-allow-methods : 허용하는 HTTP Method 목록
- access-control-allow-headers : 허용하는 헤더 목록
- access-control-max-age : preflight 요청에 대한 응답 캐싱 시간

이와 같은 과정을 통해 실제 원본 요청이 아닌 Preflight 요청으로 CORS를 검증하여 불필요한 리소스 낭비를 줄일 수 있다.

#### 트러블 슈팅

사이드 프로젝트에서 Spring Boot Interceptor를 이용해 인증 관련 부분을 구현하고 배포했다. 배포 이후에 프론트에서 요청이 제대로 처리되지 않았고 확인을 해보니 OPTIONS 요청에 401 Unauthorized 에러가 발생하는 것이었다.

<p align="center">
<img src='/assets/postImages/PreflightRequest/options-401.png' width="550" height="200">
</p>

문제 원인을 파악한 결과 CORS 검증을 위해 Preflight 요청으로 OPTIONS 요청을 보내고 있는 것을 확인했고 Interceptor에서는 해당 요청에 대해 토큰 검증을 진행하면서 401 에러를 발생시키고 있는 것이었다.

문제 원인 파악 후 Preflight 요청에 대해서 토큰 검증을 생략하게 하여 해결할 수 있었다.

```java
public class AuthenticationInterceptor {
    private static final String PREFLIGHT_METHOD = "OPTIONS";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if (PREFLIGHT_METHOD.equalsIgnoreCase(request.getMethod())) {
         
           }   return true;
    }
}
```

> [https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#the_http_response_headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#the_http_response_headers)
>
> [https://www.baeldung.com/cs/cors-preflight-requests](https://www.baeldung.com/cs/cors-preflight-requests)