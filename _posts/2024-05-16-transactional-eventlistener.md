---
layout: post
title: "@TransactionalEventListener"
categories: spring
tags: spring
---

`@TransactionalEventListener`는 `@EventListener`와는 달리 **트랜잭션 관리와 연계된 이벤트 리스너** 어노테이션이다.

> [TransactionPhase](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/event/TransactionPhase.html)
>
> `EventListener`는 `TransactionPhase`에 따라 호출이 이루어지게 된다.
>
> - AFTER_COMMIT : 커밋이 성공적으로 성공한 이후 동작
>
> - AFTER_COMPLETION : 트랜잭션이 완료된 이후 동작
>
> - AFTER_ROLLBACK : 트랜잭션이 롤백된 이후 동작
>
> - BEFORE_COMMIT : 커밋되기 전 동작

`TransactionPhase`의 디폴트 설정은 `AFTER_COMMIT`로 되어있다.

```kotlin
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@EventListener
public @interface TransactionalEventListener {
    TransactionPhase phase() default TransactionPhase.AFTER_COMMIT;

    ...
}
```

주의해야할 점은 기본 값이 `AFTER_COMMIT` 이기 때문에 별도의 설정이 필요하다.

아래 코드는 로그인 기능이고 유저를 조회 및 이벤트를 발행하여 로그인 기록을 저장하는 것을 기대할 수 있다.

```kotlin
@Transactional
fun signIn(email: String) {
    // 유저 조회
    val member = memberRepository.findByEmail(email)
        ?: throw NoSuchElementException()
    // 로그인 유저 히스토리 저장
    eventPublisher.publishEvent(SignInEvent(
        member.id,
        member.email
    ))
}

@TransactionalEventListener
fun saveMemberHistoryTransaction(event: SignInEvent) {
    log.info("invoked saveMemberHistoryTransaction")
    memberHistoryRepository.save(MemberHistory(email = event.email, member = Member(id = event.memberId, email = event.email)))
}
```

하지만, 결과를 보면 그렇지 않다. insert 쿼리가 발생하기 전에 commit이 이루어졌기 때문에 이후 쿼리는 정상 동작하지 않는다.

```sh
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : begin
[nio-8080-exec-1] org.hibernate.SQL                        : select m1_0.member_id,m1_0.email from member m1_0 where m1_0.email=?
[nio-8080-exec-1] actionalApplicationListenerMethodAdapter : Registered transaction synchronization for org.springframework.context.PayloadApplicationEvent[source=org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1338fb5, started on Wed May 15 23:44:50 KST 2024]
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : committing
[nio-8080-exec-1] m.h.event.event.MemberHistoryHandler     : invoked saveMemberHistoryTransaction
```

이를 해결하기 위해 시도할 수 있는 방법은 두 가지가 있다.

첫 번째는 `TransactionPhase`를 `BEFORE_COMMIT`로 설정하는 것이다.

```kotlin
@TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT)
fun saveMemberHistoryTransaction(event: SignInEvent) {
    log.info("invoked saveMemberHistoryTransaction")
    memberHistoryRepository.save(MemberHistory(email = event.email, member = Member(id = event.memberId, email = event.email)))
}
```

결과를 확인해 보면 insert 쿼리가 발생한 후에 커밋이 발생하여 정상 동작하는 것을 확인할 수 있다.

```sh
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : begin
[nio-8080-exec-1] org.hibernate.SQL                        : select m1_0.member_id,m1_0.email from member m1_0 where m1_0.email=?
[nio-8080-exec-1] actionalApplicationListenerMethodAdapter : Registered transaction synchronization for org.springframework.context.PayloadApplicationEvent[source=org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1338fb5, started on Wed May 15 23:48:16 KST 2024]
[nio-8080-exec-1] m.h.event.event.MemberHistoryHandler     : invoked saveMemberHistoryTransaction
[nio-8080-exec-1] org.hibernate.SQL                        : insert into member_history (email,member_member_id) values (?,?)
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : committing
```

두 번째는 리스너에서 트랜잭션 전파 속성은 `REQUIRES_NEW`로 변경하여 새로운 트랜잭션을 시작하도록 하는 것이다.

```kotlin
@TransactionalEventListener
@Transactional(propagation = Propagation.REQUIRES_NEW)
fun saveMemberHistoryTransaction(event: SignInEvent) {
    log.info("invoked saveMemberHistoryTransaction")
    memberHistoryRepository.save(MemberHistory(email = event.email, member = Member(id = event.memberId, email = event.email)))
}
```

결과를 확인해 보면 이전 트랜잭션은 커밋이 되고 새로 생성된 트랜잭션에서 insert 쿼리가 실행된 후에 커밋되어서 정상 동작하는 것을 확인할 수 있다. 

```sh
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : begin
[nio-8080-exec-1] org.hibernate.SQL                        : select m1_0.member_id,m1_0.email from member m1_0 where m1_0.email=?
[nio-8080-exec-1] actionalApplicationListenerMethodAdapter : Registered transaction synchronization for org.springframework.context.PayloadApplicationEvent[source=org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1338fb5, started on Wed May 15 23:55:28 KST 2024]
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : committing
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : On TransactionImpl creation, JpaCompliance#isJpaTransactionComplianceEnabled == false
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : begin
[nio-8080-exec-1] m.h.event.event.MemberHistoryHandler     : invoked saveMemberHistoryTransaction
[nio-8080-exec-1] org.hibernate.SQL                        : insert into member_history (email,member_member_id) values (?,?)
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : committing
```

다음으로는 `TransactionPhase`가 `AFTER_ROLLBACK`인 상황을 알아보려고 한다.

`phase`를 변경한 후 서비스 계층에서 예외를 발생시켜 일부러 롤백 상황을 유도한다.

```kotlin
@TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
fun saveMemberHistoryTransaction(event: SignInEvent) {
    log.info("invoked saveMemberHistoryTransaction")
    memberHistoryRepository.save(MemberHistory(email = event.email, member = Member(id = event.memberId, email = event.email)))
}

@Transactional
fun signIn(email: String) {
    // 유저 조회
    val member = memberRepository.findByEmail(email)
        ?: throw NoSuchElementException()
    // 로그인 유저 히스토리 저장
    eventPublisher.publishEvent(SignInEvent(
        member.id,
        member.email
    ))
    throw RuntimeException()
    // 토큰 발급
}
```

결과는 다음과 같이 롤백이 일어났지만 로그인 기록은 저장되지 않았다. 이유는 상위 레이어에서 트랜잭션이 롤백되며 종료되어 리스너에는 트랜잭션이 없기 때문이다.

```sh
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : begin
[nio-8080-exec-1] org.hibernate.SQL                        : select m1_0.member_id,m1_0.email from member m1_0 where m1_0.email=?
[nio-8080-exec-1] actionalApplicationListenerMethodAdapter : Registered transaction synchronization for org.springframework.context.PayloadApplicationEvent[source=org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1338fb5, started on Thu May 16 00:29:12 KST 2024]
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : rolling back
[nio-8080-exec-1] m.h.event.event.MemberHistoryHandler     : invoked saveMemberHistoryTransaction
[nio-8080-exec-1] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed: java.lang.RuntimeException] with root cause
```

리스너에 트랜잭션 어노테이션을 붙여 트랜잭션 내에서 동작하도록 한다.

```kotlin
@TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
@Transactional(propagation = Propagation.REQUIRES_NEW)
fun saveMemberHistoryTransaction(event: SignInEvent) {
    log.info("invoked saveMemberHistoryTransaction")
    memberHistoryRepository.save(MemberHistory(email = event.email, member = Member(id = event.memberId, email = event.email)))
}
```

롤백된 이후 새로운 트랜잭션이 시작되고 로그인 기록이 커밋되는 것을 확인할 수 있다.

```sh
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : begin
[nio-8080-exec-1] org.hibernate.SQL                        : select m1_0.member_id,m1_0.email from member m1_0 where m1_0.email=?
[nio-8080-exec-1] actionalApplicationListenerMethodAdapter : Registered transaction synchronization for org.springframework.context.PayloadApplicationEvent[source=org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1338fb5, started on Thu May 16 00:34:07 KST 2024]
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : rolling back
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : On TransactionImpl creation, JpaCompliance#isJpaTransactionComplianceEnabled == false
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : begin
[nio-8080-exec-1] m.h.event.event.MemberHistoryHandler     : invoked saveMemberHistoryTransaction
[nio-8080-exec-1] org.hibernate.SQL                        : insert into member_history (email,member_member_id) values (?,?)
[nio-8080-exec-1] o.h.e.t.internal.TransactionImpl         : committing
[nio-8080-exec-1] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed: java.lang.RuntimeException] with root cause
```

마지막으로 `@TransactionalEventListener`과 `@Transactional`을 동시에 사용할 때는 전파 속성을 `REQUIRES_NEW` 또는 `NOT_SUPPORTED`로 설정해야 한다. 

그렇게 안 하면 애플리케이션 구동 시에 리스너를 등록하는 과정에서 에러가 나게 된다.

```java
public class RestrictedTransactionalEventListenerFactory extends TransactionalEventListenerFactory {
    public RestrictedTransactionalEventListenerFactory() {
    }

    public ApplicationListener<?> createApplicationListener(String beanName, Class<?> type, Method method) {
        Transactional txAnn = (Transactional)AnnotatedElementUtils.findMergedAnnotation(method, Transactional.class);
        if (txAnn != null) {
            Propagation propagation = txAnn.propagation();
            if (propagation != Propagation.REQUIRES_NEW && propagation != Propagation.NOT_SUPPORTED) {
                throw new IllegalStateException("@TransactionalEventListener method must not be annotated with @Transactional unless when declared as REQUIRES_NEW or NOT_SUPPORTED: " + method);
            }
        }

        return super.createApplicationListener(beanName, type, method);
    }
}
```