---
layout: post
title: Feign Client
categories: spring
tags: spring
---

Feign은 Java 라이브러리로 HTTP 클라이언트를 쉽게 작성할 수 있게 도와준다. 안드로이드에서 인터페이스 형태로 작성하여 HTTP Client로 사용되는 Retrofit에서 영감을 받아 HTTP API와 Java 코드 간의 바인딩을 간소화하여 쉽게 연결할 수 있게 해주는 것이 주된 목적이다.

**Dependency**

```gradle
dependencies {
    implementation("org.springframework.cloud:spring-cloud-starter-openfeign")
}
```

**Feign Client Interceptor**

헤더 공통 설정을 위해 api token을 포함시킬 인터셉터를 구현한다. 만약 개별로 헤더 값을 지정하고 싶다면 인터페이스에서 `@Headers` 를 사용해도 된다.

```kotlin
// application.yml
openai:
  api:
    url: "https://api.openai.com"
    key: "token"

class OpenAiHeaderConfig {

    @Value("\${openai.api.key}")
    lateinit var apiKey: String

    @Bean
    fun requestInterceptor(): RequestInterceptor {
        return RequestInterceptor { requestTemplate ->
            requestTemplate.header("Authorization", "Bearer $apiKey")
        }
    }
}
```

**Feign client**

openai와 통신하기 위해 base url와 인터셉터를 설정하고 인터페이스를 정의한다.

```kotlin
@FeignClient(name = "openai", url = "\${openai.api.url}", configuration = [OpenAiHeaderConfig::class])
interface OpenAiClient {

    @PostMapping("/v1/chat/completions")
    fun chat(@RequestBody request: OpenAiDto.OpenAiChatRequest): OpenAiDto.OpenAiChatResponse
}

class OpenAiDto {
    data class OpenAiChatRequest(
        val model: String,
        val messages: List<ChatMessage>
    )
    
    data class OpenAiChatResponse(
        val id: String,
        val `object`: String,
        val created: Long,
        val choices: List<Choice>,
        val usage: Usage
    )
    ...
}
```

**Feign Client 활성화 및 로그 설정**

`@EnableFeignClients`를 사용하여 지정된 패키지에 feign client를 찾아서 bean으로 등록한다. 그리고 로그 확인을 위해 로깅 설정도 같이 작성한다. 자세한 관련 내용은 [Feign logging](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/#feign-logging)를 참고하면 된다.

```kotlin
@Configuration
@EnableFeignClients(basePackages = ["me.hajoo"])
class OpenAiFeignConfig {

    @Bean
    fun feignLoggerLevel(): Logger.Level {
        return Logger.Level.FULL;
    }
}
```

**테스트**

```kotlin
@SpringBootTest
class OpenAiFeignClientTest(
    @Autowired val openAiClient: OpenAiClient,
) {

    @Test
    fun chatTest() {
        val response = openAiClient.chat(
            OpenAiDto.OpenAiChatRequest(
                model = "gpt-4o",
                messages = listOf(
                    OpenAiDto.ChatMessage(
                        role = "user",
                        content = "Hello World!"
                    )
                )
            )
        )
        println(response)
    }
}
```

> [https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/#spring-cloud-feign](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/#spring-cloud-feign)
>
> [https://github.com/OpenFeign/feign](https://github.com/OpenFeign/feign)
> 
> [https://techblog.woowahan.com/2630/](https://techblog.woowahan.com/2630/)