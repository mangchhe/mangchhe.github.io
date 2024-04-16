---
layout: post
title: When allowCredentials is true, allowedOrigins cannot contain the special value "*"
categories: spring
tags: spring
---

#### 에러 메시지

> When allowCredentials is true, allowedOrigins cannot contain the special value "*" since that cannot be set on the "Access-Control-Allow-Origin" response header. To allow credentials to a set of origins, list them explicitly or consider using "allowedOriginPatterns" instead.

#### 문서

> [setAllowedOriginPatterns](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/cors/CorsConfiguration.html#setAllowedOriginPatterns(java.util.List))
>
> In contrast to allowedOrigins which only supports "*" and cannot be used with allowCredentials or allowPrivateNetwork, when an allowedOriginPattern is matched, the Access-Control-Allow-Origin response header is set to the matched origin and not to "*" nor to the pattern.

allowCredentials 가 true일 경우에는 '\*' 값을 response header(`Access-Control-Allow-Origin`)에 추가할 수 없어 `allowedOrigins` 값에 '\*' 을 값을 포함할 수 없다고 한다. 대신 `allowOriginPatterns`를 사용하면 해결된다고 나와있다.

```kotlin
@Configuration
class CorsConfig {
    @Bean
    fun corsFilter() : CorsFilter {
        val source = UrlBasedCorsConfigurationSource()
        val config = CorsConfiguration()
        config.allowCredentials = true
        // config.allowedOrigins("*")
        config.addAllowedOriginPattern("*")
        config.addAllowedHeader("*")
        config.addAllowedMethod("*")
        source.registerCorsConfiguration("/**", config)
        return CorsFilter(source)
    }
}
```