---
layout: post
title: AWS CloudWatch in Spring Boot
categories: aws
tags: aws
---

## `build.gradle`

```gradle
implementation("ca.pjer:logback-awslogs-appender:1.6.0")
```

## `logback-spring.yml`

```xml
<configuration>
    <appender name="CLOUDWATCH_LOGGING" class="ca.pjer.logback.AwsLogsAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <!-- Nice layout pattern -->
        <layout>
            <pattern>[%p][%X{traceId}][%thread] - %msg%n</pattern>
        </layout>
        <!-- Hardcoded Log Group Name -->
        <logGroupName>testLogGroupName</logGroupName>
        <!-- Log Stream Name UUID Prefix -->
        <logStreamUuidPrefix>test-/</logStreamUuidPrefix>
        <!-- Hardcoded AWS region -->
        <logRegion>ap-northeast-2</logRegion>
        <!-- Maximum number of events in each batch (50 is the default) -->
        <!-- will flush when the event queue has 50 elements, even if still in quiet time (see maxFlushTimeMillis) -->
        <maxBatchLogEvents>50</maxBatchLogEvents>
        <!-- Maximum quiet time in millisecond (0 is the default) -->
        <!-- will flush when met, even if the batch size is not met (see maxBatchLogEvents) -->
        <maxFlushTimeMillis>30000</maxFlushTimeMillis>
        <!-- Maximum block time in millisecond (5000 is the default) -->
        <!-- when > 0: this is the maximum time the logging thread will wait for the logger, -->
        <!-- when == 0: the logging thread will never wait for the logger, discarding events while the queue is full -->
        <maxBlockTimeMillis>5000</maxBlockTimeMillis>
        <!-- Retention value for log groups, 0 for infinite see -->
        <!-- https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_PutRetentionPolicy.html for other -->
        <!-- possible values -->
        <retentionTimeDays>0</retentionTimeDays>
        <!-- Use custom credential instead of DefaultCredentialsProvider -->
        <accessKeyId>testAccessKey</accessKeyId>
        <secretAccessKey>testSecretAccessKey</secretAccessKey>
    </appender>
    <root level="INFO">
        <appender-ref ref="CLOUDWATCH_LOGGING">
    </root>
</configuration>
```

> [https://github.com/pierredavidbelanger/logback-awslogs-appender](https://github.com/pierredavidbelanger/logback-awslogs-appender)

IAM을 사용하여 **CloudWatchFullAccess** 권한을 가진 유저를 만들어서 따로 키를 생성하는 것이 좋다. 항상 키는 최소한의 권한을 유지하도록 하자.

AWS Console 에서 CloudWatch → 로그 → 로그 그룹 → 로그 스트림 에서 로그가 쌓이는 것을 확인할 수 있다.