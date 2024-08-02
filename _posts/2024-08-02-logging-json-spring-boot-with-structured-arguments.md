---
layout: post
title: Logging JSON in Spring Boot with StructuredArguments
categories: spring
tags: spring
---

시스템 운영에서 중요하다고 느낀 부분은 로그 데이터이다. 사전에 확립된 일관되고 정형화된 형식의 로그는 오류를 더 빠르게 파악하는 데 큰 도움을 주었다.

로그 데이터의 형식이 구조화되지 않으면 서드파티에서 데이터를 추출하기 어렵고 가독성이 떨어져 관리하기 힘들어진다. 또한, 오류 지점을 찾는 데 더 많은 시간이 소요된다.

이번에 Datadog에 수집되는 로그를 개선하기 위해 로그 파악이 쉬운 구조로 포맷 형식(json)을 변경하는 작업을 진행했다.

`logstash logback encoder`를 이용하면 로그를 간단하게 json 형식으로 변환할 수 있다.

```java
implementation("net.logstash.logback:logstash-logback-encoder:7.4")
```

```java
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

다음과 같이 세팅하게 되면 기본적으로 로그 아래와 같이 나타나게 된다.

```
{
    "@timestamp": "2024-08-02T22:25:06.752828+09:00",
    "@version": "1",
    "message": "Hello World!",
    "logger_name": "me.hajoo.TestController",
    "thread_name": "http-nio-8080-exec-1",
    "level": "INFO",
    "level_value": 20000
}
```

**Custom Json Key-Value 추가**

custom field 추가 하는 방식으로 세 가지 방식이 사용할 수 있고 MDC는 값이 문자열 형태로 들어가기 때문에 2 depth 이상의 json format으로는 만들 수 없기 때문에 depth가 있는 object를 json 형식으로 구조화하기 위해서는 MDC 외에 방식을 사용해야 한다.

```kotlin
// MDC
MDC.put("key1", a.toString())
log.info("Hello World!")
// log.atInfo()
log.atInfo().addKeyValue("key1", a).addKeyValue("key2", a).log();
// StructuredArguments
// kv(String key, Object value) : object to json format
// raw(String fieldName, String rawJsonValue) : json string to json format
log.info("Hello World!", StructuredArguments.kv("key1", a), StructuredArguments.raw("key2", jsonStr));
```

> [https://www.baeldung.com/java-structured-logging](https://www.baeldung.com/java-structured-logging)