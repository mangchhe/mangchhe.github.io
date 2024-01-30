---
layout: post
title: Set MDC(TraceId) in Webflux
categories: webflux
tags: webflux
---

한 트랜잭션 내에서 로그 트레이싱을 쉽게 하기 위해서 `traceId`를 사용하여 연관된 로그를 이어준다.

```kotlin
object Contexts {
    const val TRACE_ID = "traceId"
}

@Component
class MdcFilter : WebFilter, Ordered {

    override fun getOrder(): Int {
        return Ordered.HIGHEST_PRECEDENCE; // 다른 필터들 보다 먼저 실행될 수 있도록 우선순위를 가장 높게 설정
    }

    override fun filter(exchange: ServerWebExchange, chain: WebFilterChain): Mono<Void> {
        return chain.filter(exchange)
            .contextWrite(Context.of(Contexts.TRACE_ID, UUID.randomUUID().toString())) // 랜덤으로 생성한 UUID를 Context에 저장
    }
}
```

## Reactor with MDC

일부 로그 관련 메타 데이터를 저장하기 위해 MDC에 저장하여 사용해 왔다. 하지만 Webflux 는 비동기 논블럭킹 처리 방식으로 한 요청이 여러 스레드가 처리될 수 있다. 그렇기 때문에 ThreadLocal을 사용하는 MDC에 저장하는 방식은 제대로 동작하지 않기 때문에 별도의 설정이 필요하다.

`Subscriber`를 구현하여 `onNext`(데이터 발행), `onError`(예외 발생) 시에 Context에 저장되어 있는 `traceId`를 가져와 현재 스레드의 MDC에 저장하도록 구현한다.

```kotlin
class MdcContextLifter<T>(private val coreSubscriber: CoreSubscriber<T>) : CoreSubscriber<T> {

    override fun onSubscribe(s: Subscription) {
        coreSubscriber.onSubscribe(s)
    }

    override fun onNext(t: T) {
        currentContext().copyContextMapToMdc()
        coreSubscriber.onNext(t)
    }

    override fun onError(t: Throwable?) {
        currentContext().copyContextMapToMdc()
        coreSubscriber.onError(t)
    }

    override fun onComplete() {
        coreSubscriber.onComplete()
    }

    override fun currentContext() = coreSubscriber.currentContext()

    private fun Context.copyContextMapToMdc() {
        if (!isEmpty) {
            getOrEmpty<String>(Contexts.TRACE_ID)
                .ifPresent {
                    MDC.put(Contexts.TRACE_ID, it)
                }
        }
    }
}
```

Reactor의 `Hooks.onEachOperator`를 사용하여 각 오퍼레이터에 훅을 등록하여 특정 작업을 수행할 수 있게 하고 위에 구현한 `Subscriber`를 오퍼레이터 체인에 새로운 동작으로 주입하게 되면 `traceId`가 이어지는 것을 확인할 수 있다.

```kotlin
@Configuration
class MdcContextLifterConfig {

    val mdcContextReactorKey: String = MdcContextLifterConfig::class.java.name

    @PostConstruct
    fun contextOperatorHook() = Hooks.onEachOperator(
        mdcContextReactorKey,
        Operators.lift { _, subscriber -> MdcContextLifter(subscriber) }
    )
}
```