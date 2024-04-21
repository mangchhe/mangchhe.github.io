---
layout: post
title: Logging Request & Response in Spring Boot
categories: spring
tags: spring
---

서비스를 운영할 때, 요청과 응답을 로깅하는 것은 매우 중요하다. 이를 통해 나중에 문제가 발생했을 때 디버깅에 유용한 정보를 수집하거나 성능을 측정하고 데이터를 수집할 수 있기 때문이다.

아래와 같이 AOP를 활용하여 로깅을 구현한다. (필터, 인터셉터를 이용하는 방법도 존재)

```kotlin
@Aspect
@Component
class LoggingConfig {

    private val logger = LoggerFactory.getLogger(this.javaClass)

    @Pointcut("""
        within(me.hajoo..*)
        &&
        @within(org.springframework.web.bind.annotation.RestController)
        """
    )
    fun loggingPointcut() {
    }

    @Around("loggingPointcut()")
    fun logging(joinPoint: ProceedingJoinPoint): Any? {
        val requestAttributes = RequestContextHolder.currentRequestAttributes() as ServletRequestAttributes
        val request: HttpServletRequest = requestAttributes.request

        val requestInfo = mutableMapOf<String, Any>()
        requestInfo["Method"] = request.method
        requestInfo["Url"] = request.requestURL
        requestInfo["Ip"] = request.remoteAddr

        val headers = Collections.list(request.headerNames)
            .associateWith { headerName -> Collections.list(request.getHeaders(headerName)) }
        requestInfo["Headers"] = headers

        val objectMapper = ObjectMapper()

        val requestBody = joinPoint.args.firstOrNull { it is Any && it !is HttpServletRequest }
        if (requestBody != null) {
            requestInfo["Body"] = objectMapper.writeValueAsString(requestBody)
        }

        val requestInfoJson = objectMapper.writeValueAsString(requestInfo)
        logger.info(requestInfoJson)

        val response = try {
            joinPoint.proceed()
        } catch (ex: Throwable) {
            requestInfo["Exception"] = ex.message ?: "Unknown exception"
            throw ex
        }

        val responseInfo = mutableMapOf<String, Any>()
        responseInfo["Response"] = response ?: ""

        val responseInfoJson = objectMapper.writeValueAsString(responseInfo)
        logger.info(responseInfoJson)

        return response
    }
}
```

요청과 응답을 로깅하는 것은 중요하지만, 로깅하는 데이터의 양을 고려해야 한다. 과도한 로깅은 비용을 초래할 수 있기 때문에 필요한 정보에 중점을 두는 것이 좋다.

코드에서는 나타나지 않았지만, 추적을 개선하기 위해 `traceId`와 `spanId`를 함께 포함하는 것이 좋다.

```kotlin
@RestController
class UserController {

    @PostMapping("sign-in")
    fun signIn(@RequestBody request: SignIn.Request): SignIn.Response {
        return SignIn.Response(request.name)
    }
}

class SignIn {
    data class Request (
        val name: String
    )

    data class Response (
        val name: String
    )
}
```

API 테스트를 진행하면 아래와 같이 로깅되어 나오는 걸 볼 수 있다.

**Console**

```java
2024-04-15T21:46:21.108+09:00  INFO 88845 --- [logging-aop] [nio-8080-exec-1] m.hajoo.loggingaop.config.LoggingConfig  : {"Method":"POST","Url":"http://localhost:8080/sign-in","Ip":"127.0.0.1","Headers":{"content-type":["application/json"],"content-length":["21"],"host":["localhost:8080"],"connection":["Keep-Alive"],"user-agent":["Apache-HttpClient/4.5.14 (Java/17.0.9)"],"accept-encoding":["br,deflate,gzip,x-gzip"]},"Body":"{\"name\":\"hajoo\"}"}
2024-04-15T21:46:21.108+09:00  INFO 88845 --- [logging-aop] [nio-8080-exec-1] m.hajoo.loggingaop.config.LoggingConfig  : {"Response":{"name":"hajoo"}}
```

**Prettier**

```json
{
    "Method": "POST",
    "Url": "http://localhost:8080/sign-in",
    "Ip": "127.0.0.1",
    "Headers": {
        "content-type": [
            "application/json"
        ],
        "content-length": [
            "21"
        ],
        "host": [
            "localhost:8080"
        ],
        "connection": [
            "Keep-Alive"
        ],
        "user-agent": [
            "Apache-HttpClient/4.5.14 (Java/17.0.9)"
        ],
        "accept-encoding": [
            "br,deflate,gzip,x-gzip"
        ]
    },
    "Body": "{\"name\":\"hajoo\"}"
}

{
    "Response": {
        "name": "hajoo"
    }
}
```

로그를 남기고 싶지 않은 API도 있는 경우 어노테이션을 만들어서 로그에서 제외할 수 있다.

`ExcludeLog` 어노테이션으로 만들고 메서드에 해당 어노테이션이 없을 경우에만 AOP를 로직을 거칠 수 있게 설정 후 제외하는 싶은 API에 어노테이션을 달아주면 된다.

```kotlin
// ExcludeLog.kt
@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.FUNCTION)
annotation class ExcludeLog

// LoggingConfig.kt
@Pointcut("@annotation(me.hajoo.loggingaop.annotation.ExcludeLog)")
fun excludeLogPointcut() {}

@Pointcut("""
    (within(me.hajoo..*)
    &&
    @within(org.springframework.web.bind.annotation.RestController))
    && !excludeLogPointcut()
    """
)
```

아래와 같이 로깅이 필요 없는 메서드에 어노테이션을 달아 제외시킨다.

```kotlin
@RestController
class UserController {

    @PostMapping("sign-in")
    fun signIn(@RequestBody request: SignIn.Request): SignIn.Response {
        return SignIn.Response(request.name)
    }

    @ExcludeLog
    @GetMapping("profile")
    fun getProfile() {
    }
}
```

만약에 로그만 안 남기는 거고 AOP를 거쳐 일부 로직을 진행해야 하는 경우에는 리플렉션을 이용하여 메서드에 어노테이션을 추출하여 일부 로직만 분기 처리하면 된다.

```kotlin
@Aspect
@Component
class LoggingConfig {
    @Around("loggingPointcut()")
    fun logging(joinPoint: ProceedingJoinPoint): Any? {
        ...

        // 메서드에서 어노테이션 추출
        val methodSignature = joinPoint.signature as MethodSignature
        val method = methodSignature.method
        val excludeLogAnnotation = method.getAnnotation(ExcludeLog::class.java)

        val requestInfoJson = objectMapper.writeValueAsString(requestInfo)
        // 어노테이션이 없는 경우에만 로깅
        excludeLogAnnotation?: logger.info(requestInfoJson)

        val response = try {
            joinPoint.proceed()
        } catch (ex: Throwable) {
            requestInfo["Exception"] = ex.message ?: "Unknown exception"
            throw ex
        }

        val responseInfo = mutableMapOf<String, Any>()
        responseInfo["Response"] = response

        val responseInfoJson = objectMapper.writeValueAsString(responseInfo)
        // 어노테이션이 없는 경우에만 로깅
        excludeLogAnnotation?: logger.info(responseInfoJson)

        return response
    }
}
```