---
layout: post
title: EnvironmentPostProcessor
categories: spring
tags: spring
---

> [EnvironmentPostProcessor](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/env/EnvironmentPostProcessor.html)
>
> Allows for customization of the application's Environment prior to the application context being refreshed.
>
> 애플리케이션 Environment를 커스터마이징할 수 있게 허락한다.

> [Environment](https://docs.spring.io/spring-framework/docs/6.1.6/javadoc-api/org/springframework/core/env/Environment.html)
>
> Interface representing the environment in which the current application is running. Models two key aspects of the application environment: profiles and properties.
>
> 현재 실행 중인 애플리케이션의 Environment를 나타내고 주요 측면으로 profile 과 properties를 모델링한다.

`EnvironmentPostProcessor` 인터페이스를 통해 애플리케이션 구동 시에 원하는 방식대로 설정 파일(yaml, properties) 또는 profile을 설정할 수 있다.

### spring.profiles.include

설정 파일을 application-redis.yml, application-mysql.yml 다음과 같이 분리하여 application.yml에서 include 하여 사용하는 방식이 있다.

```yml
// application.yml

spring:
    profiles:
        include:
            - redis
            - mysql
```

멀티 모듈에서 다음과 같이 사용할 때 불편한 점이 존재한다. common이나 client 모듈에서 사용하는 설정 파일을 생성할 때 app 모듈로 와서 application.yml에 include 해주어야 하고 새로운 app 모듈이 생성되었을 때 필요한 부분을 copy & paste 해야 하는 번거로움이 존재한다.

설정 파일이 여러 모듈에 중복되면 동일한 설정을 여러 곳에서 반복해서 수정해야 하고 새로운 모듈 추가 시에 기존 모듈 설정을 복사해야 하는 등 관리하기 어려워질 것이고 모듈의 개수가 많아질수록 더욱더 체감될 것이다.

이러한 문제들은 설정 일관성을 무너뜨리고 유지보수의 어려움이 발생된다.

### EnvironmentPostProcessor 구현

`EnvironmentPostProcessor` 를 이용하여 include 하지 않고 yaml 설정 파일들을 읽어 적용하는 방식을 적용하면 확장하는 데 있어 유연하게 대처할 수 있다.

```kotlin
class HjEnvironmentPostProcessor : EnvironmentPostProcessor {
    
    companion object {
        private const val FILE_PATH = "classpath*:"
        private const val FILE_PATTERN = "application-*"
        private val YAML_EXTENSIONS = listOf(".yml", ".yaml")
    }

    override fun postProcessEnvironment(environment: ConfigurableEnvironment, application: SpringApplication) {
        val sources = environment.propertySources
        val resolver = PathMatchingResourcePatternResolver()

        try {
            val resources = mutableListOf<Resource>()
            YAML_EXTENSIONS.forEach {
                resources.addAll(resolver.getResources(FILE_PATH + FILE_PATTERN + it))
            }
            resources.forEach { resource ->
                YamlPropertySourceLoader().load(resource.filename, resource)
                    .forEach {
                        sources.addLast(it)
                    }
            }
        } catch (e: Exception) {
            throw RuntimeException("Failed to load configuration files", e)
        }
    }
}
```

```kotlin
// path: src.main.resources.META-INF
// file name: spring.factories

org.springframework.boot.env.EnvironmentPostProcessor=\
me.hajoo.hj.yaml.importer.HjEnvironmentPostProcessor
```