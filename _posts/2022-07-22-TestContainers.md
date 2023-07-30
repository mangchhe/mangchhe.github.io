---
layout: post
title: "TestContainers"
categories: test
tags: test
---

- Junit Test를 지원하는 Java Library로, Docker 컨테이너를 사용할 수 있도록 일회용 인스턴스를 제공한다.
- 일회용 Docker Containers를 이용하여 독립적이고 멱등성 있는 테스트 환경을 구축할 수 있다.

## 기존 테스트 방식

- 테스트하기 위해서 우리는 다양한 방법을 사용한다.
- H2와 같은 인메모리 디비를 사용하거나 Docker를 Local에 띄우거나 테스트 환경 DB를 따로 구축해서 사용한다.
- 하지만, 각각은 문제점들이 존재한다.
    - 인메모리 디비는 실제 운영 DB 벤더와의 쿼리에서 차이점이 생길 수 있다.
    - Docker를 local에 띄워서 하는 것은 개발자마다 서로 다른 테스트 데이터를 가지게 되고 계속 local에 띄우기 때문에 그만큼 메모리를 차지하게 된다.
    - 테스트 환경 DB는 따로 테스트 환경 DB를 띄워야 하는 리소스와 테스트 데이터를 서로 추가하다 보면 유지보수가 어려워질 수 있다.

## 사용했을 때 이점

- 데이터베이스 벤더에 맞는 Docker 컨테이너를 띄워서 사용하기 때문에 쿼리 사용과 발생에 있어 실제 DB와 차이점이 없다.
- 테스트할 때만 Docker 컨테이너를 띄우기 때문에 계속해서 메모리를 차지하지 않는다.
- 테스트 환경 DB를 따로 만들지 않기 때문에 추가 리소스가 필요하지 않다.
- 매번 테스트할 데이터들이 동일하기 때문에 멱등성 있는 테스트가 보장되며 테스트 데이터를 추가하기 위해서는 초기 데이터 설정 파일을 변경해야 때문에 좀 더 조심스럽고 안전하게 관리될 수 있다.

## 단점

- 테스트를 실행할 때마다 도커 컨테이너를 생성하고 세팅하는 작업이 추가되게 된다.
- 아무리 가벼운 테스트라고 할지라도 도커 컨테이너를 생성하고 세팅하는 시간이 있어 시간이 좀 더 소요되게 된다.

# 실습

## Dependency

```s
// TestContainers
testImplementation "org.junit.jupiter:junit-jupiter:5.8.1"
testImplementation "org.testcontainers:testcontainers:1.17.2"
testImplementation "org.testcontainers:junit-jupiter:1.17.2"

// Mysql TestContainers
testImplementation "org.testcontainers:mysql:1.17.2"
```

## Java Config

- `@Testcontainers`, `@Container`와 함께 `new MySQLContainer<>("mysql:8.0.24")` 사용할 이미지를 세팅한다.
    - 데이터베이스 벤더별 사용법 [참고](https://www.testcontainers.org/modules/databases/)
- `withInitScript("schema.sql")`는 도커 컨테이너를 생성하고 초기 설정을 실행할 .sql 파일을 지정한다.
- 주의할 점, 컨테이너 객체 생성 시에 static을 붙여 정적으로 생성하지 않으면 테스트 메서드별로 도커 컨테이너를 생성하여 실행한다.

```java
@SpringBootTest
@Testcontainers
public class JavaConfigTest {

    @Container
    private static final MySQLContainer<?> MY_SQL_CONTAINER = new MySQLContainer<>("mysql:8.0.24")
            .withInitScript("schema.sql");

    @DynamicPropertySource
    public static void properties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", MY_SQL_CONTAINER::getJdbcUrl);
        registry.add("spring.datasource.username", MY_SQL_CONTAINER::getUsername);
        registry.add("spring.datasource.password", MY_SQL_CONTAINER::getPassword);
    }
}
```

## YML Config

- `jdbc:tc:{docker image}:///{db name}`로 url을 설정하면 된다.
- 초기 데이터를 세팅하기 위해서는 `?TC_INITSCRIPT={.sql file}`를 추가하면 된다.

```yml
spring:
  datasource:
    driver-class-name: org.testcontainers.jdbc.ContainerDatabaseDriver
    url: jdbc:tc:mysql:8.0.24:///test?TC_INITSCRIPT=schema.sql
```

### Reference

- [https://www.testcontainers.org/](https://www.testcontainers.org/)