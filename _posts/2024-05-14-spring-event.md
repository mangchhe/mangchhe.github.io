---
layout: post
title: Spring Event
categories: spring
tags: spring
---

#### 기존 코드

로그인 코드는 로그인 기능과 히스토리 저장 기능이 함께 포함되어 있어 서로 강하게 결합되어 있다.

```kotlin
fun signIn(email: String) {
    // 유저 조회
    val member = memberRepository.findByEmail(email)
        ?: throw NoSuchElementException()

    // 로그인 유저 히스토리 저장
    memberHistoryRepository.save(
        MemberHistory(
            email = email
        )
    )
    // 토큰 발급
}
```

#### Spring Event를 활용한 관심사 분리

**이벤트 클래스 정의**

이벤트 클래스를 정의한다. 해당 클래스는 이벤트가 발생할 때 전달할 데이터를 담고 있다.

```kotlin
class SignInEvent(val email: String)
```

**이벤트 핸들러 정의**

이벤트 핸들러는 이벤트가 발생했을 때 실행될 로직을 담고 있다. 여기에서 로그인 히스토리 저장하는 로직을 구현하고 이벤트 리스너로 등록한다.

```kotlin
@Component
class MemberHistoryHandler(
    private val memberHistoryRepository: MemberHistoryRepository,
) {

    @EventListener
    fun saveMemberHistory(event: SignInEvent) {
        memberHistoryRepository.save(MemberHistory(email = event.email))
    }
}
```

**기존 코드 수정**

로그인 히스토리 저장하는 로직을 제거하고 이벤트를 발행한다.

```kotlin
@Service
class MemberService(
    private val memberRepository: MemberRepository,
    private val eventPublisher: ApplicationEventPublisher,
) {

    fun signIn(email: String) {
        // 유저 조회
        val member = memberRepository.findByEmail(email)
            ?: throw NoSuchElementException()
        // 로그인 유저 히스토리 저장
        eventPublisher.publishEvent(SignInEvent(email))
        // 토큰 발급
    }
}
```

**비즈니스 로직 변경**

만약 로그인 시에 로그인 메시지를 보내야하는 요구 사항이 생겼다면 리스너만 새로 구현해서 등록하면 되기 때문에 기존 비즈니스 코드를 수정할 필요가 없다.

```kotlin
@Component
class SendMessageHandler(
) {

    @EventListener
    fun sendMessage(event: SignInEvent) {
        println("You are logged in.")
    }
}
```

**Spring Event**를 활용하면 다음과 같은 장점이 있다.

- 관심사의 분리: 로그인 기능과 히스토리 저장 기능이 서로 분리되어 코드의 가독성과 유지보수성이 향상
- 느슨한 결합: 각 기능이 독립적으로 작동하므로, 하나의 기능을 변경해도 다른 기능에 영향 X
- 유연한 확장성: 새로운 기능을 쉽게 추가할 수 있으며, 기존 코드를 수정할 필요 없이 확장

#### 디버깅

**리스너 등록**

애플리케이션 구동 시에 `applicationListeners` 컬렉션에 리스너가 등록된다.

```java
// AbstractApplicationEventMulticaster:276

private class DefaultListenerRetriever {
    public final Set<ApplicationListener<?>> applicationListeners = new LinkedHashSet();
    ...
}

// AbstractApplicationEventMulticaster:76

public void addApplicationListener(ApplicationListener<?> listener) {
    synchronized(this.defaultRetriever) {
        Object singletonTarget = AopProxyUtils.getSingletonTarget(listener);
        if (singletonTarget instanceof ApplicationListener) {
            this.defaultRetriever.applicationListeners.remove(singletonTarget);
        }

        this.defaultRetriever.applicationListeners.add(listener);
        this.retrieverCache.clear();
    }
}
```

**이벤트 발행**

`ApplicationEventPublisher` 에서 `publishEvent` 함수 호출 시에 `AbstractApplicationContext` 에 `multicastEvent`를 실행하게 된다.

```java
// AbstractApplicationContext:219

if (this.earlyApplicationEvents != null) {
    this.earlyApplicationEvents.add(applicationEvent);
} else if (this.applicationEventMulticaster != null) {
    this.applicationEventMulticaster.multicastEvent((ApplicationEvent)applicationEvent, eventType);
}
```

`getApplicationListeners`에서 동일한 이벤트 타입의 리스너를 가져와 `while` 문에서 순환하여 관련 리스너가 실행되게 된다.

```java
public void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType) {
    ResolvableType type = eventType != null ? eventType : ResolvableType.forInstance(event);
    Executor executor = this.getTaskExecutor();
    Iterator var5 = this.getApplicationListeners(event, type).iterator();

    while(true) {
        while(var5.hasNext()) {
            ApplicationListener<?> listener = (ApplicationListener)var5.next();
            if (executor != null && listener.supportsAsyncExecution()) {
                try {
                    executor.execute(() -> {
                        this.invokeListener(listener, event);
                    });
                } catch (RejectedExecutionException var8) {
                    this.invokeListener(listener, event);
                }
            } else {
                this.invokeListener(listener, event);
            }
        }

        return;
    }
```