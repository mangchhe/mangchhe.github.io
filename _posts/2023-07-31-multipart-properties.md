---
layout: post
title: Multipart Properties
categories: spring
tags: spring
---

```java
@ConfigurationProperties(
    prefix = "spring.servlet.multipart",
    ignoreUnknownFields = false
)
public class MultipartProperties {
    private boolean enabled = true;
    private String location;
    private DataSize maxFileSize = DataSize.ofMegabytes(1L);
    private DataSize maxRequestSize = DataSize.ofMegabytes(10L);
    private DataSize fileSizeThreshold = DataSize.ofBytes(0L);
    ...
}
```

## 공식 문서

> 
**location** specifies the directory where uploaded files will be stored. When not specified, a temporary directory will be used.
>
**max-file-size** specifies the maximum size permitted for uploaded files. The **default is 1MB**
>
**max-request-size** specifies the maximum size allowed for multipart/form-data requests. The **default is 10MB.**
>
**file-size-threshold** specifies the size threshold after which files will be written to disk. The **default is 0.**

- location : 파일이 저장될 위치. 디폴트는 임시 디렉터리
- max-file-size : 업로드 파일 크기. 디폴트는 1MB
- max-request-size : multipart/form-data content-type의 허용 요청 크기. 디폴트는 10MB
- file-size-threshold : 디스크에 저장될 파일의 임계치. 디폴트는 0Byte

## Properties 설정

```yml
spring:
    servlet:
        multipart:
            location: classpath:upload-files
            max-file-size: 10MB
            max-request-size: 20MB
            file-size-threshold: 2097152
```


### Reference

- [MultipartProperties 공식 문서](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/web/servlet/MultipartProperties.html)