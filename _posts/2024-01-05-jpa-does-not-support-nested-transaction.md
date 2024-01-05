---
layout: post
title: JPA itself does not support nested transactions
categories: jpa
tags: jpa
---

> [JpaTransactionManager](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/orm/jpa/JpaTransactionManager.html)
> 
> **Note that JPA itself does not support nested transactions!** Hence, do not expect JPA access code to semantically participate in a nested transaction.

## 요구사항

- 사용자를 등록할 때 히스토리를 같이 저장한다.
- 제약 조건
  - 사용자 등록에 실패하면 히스토리 저장도 하지 않는다.   
  - 사용자 등록에 성공했지만 히스토리 저장에 실패한다면 사용자 등록은 그대로 진행한다.

## 소스 코드

```kotlin
@Configuration
@EnableTransactionManagement
class Config {
    @Bean
    fun transactionManager(entityManagerFactory: EntityManagerFactory): JpaTransactionManager {
        return JpaTransactionManager(entityManagerFactory)
    }
}

@Service
class OuterService(
    private val memberRepository: MemberRepository,
    private val innerService: InnerService,
) {
    @Transactional
    fun saveMember() {
        memberRepository.save(Member(name = "tester"))
        try {
        innerService.saveMemberHistory()
        } catch (ex: Exception) {}
    }
}

@Service
class InnerService(
    private val memberHistoryRepository: MemberHistoryRepository,
) {
    @Transactional(propagation = Propagation.NESTED)
    fun saveMemberHistory() {
        memberHistoryRepository.save(MemberHistory(name = "tester"))
        throw RuntimeException()
    }
}
```

## 결과

- `saveMember`에서 부모 트랜잭션이 시작되고 `saveMemberHistory` 에서 자식 트랜잭션이 시작되는데 예외가 발생하여 `saveMemberHistory`의 트랜잭션만 롤백되기를 기대하지만 그렇게 동작하지 않는다.
- `NestedTransactionNotSupportedException` 익셉션을 발생시키는 것을 확인할 수 있다.

```kotlin 
org.springframework.transaction.NestedTransactionNotSupportedException: JpaDialect does not support savepoints - check your JPA provider's capabilities
```