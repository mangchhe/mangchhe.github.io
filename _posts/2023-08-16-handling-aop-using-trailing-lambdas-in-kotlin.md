---
layout: post
title: Handling AOP using Trailing Lambdas in Kotlin
categories: spring
tags: spring
---

테크 블로그 [Kotlin으로 Spring AOP 극복하기!](https://tech.kakaopay.com/post/overcome-spring-aop-with-kotlin/)를 토대로 작성.

> 
**[Passing Trailing Lambdas](https://kotlinlang.org/docs/lambdas.html#passing-trailing-lambdas)**
>
According to Kotlin convention, if the last parameter of a function is a function, then a lambda expression passed as the corresponding argument can be placed outside the parentheses:

만약 함수의 마지막 파라미터가 함수이면, 인자로 전달된 람다 표현식을 괄호 바깥에 배치할 수 있다.

```kotlin
val product = items.fold(1) { acc, e -> acc * e }
```

## AOP 아쉬운 점

**구현의 번거로움**

어노테이션 생성, Advice 정의한 클래스 구현(Pointcut에서 사용하는 SpEL 등 익숙하지 않은 문법 사용)에 있어서 번거롭다.

**내부 함수 호출 적용 불가**

같은 클래스의 메서드를 호출할 경우 프록시를 타지 않기 때문에 프록시 객체를 통해 메서드를 불러주거나 다른 클래스에 만들어서 호출해야 한다.

**런타임 예외 발생 가능성**

포인트 컷 표현식이 의도한 대로 적용이 되었는지, JoinPoint의 인자의 순서와 타입이 일치한 지 컴파일 시점에 알 수 있는 방법은 없다.

## 구현

`Trailing Lambdas` 문법을 사용하여 걸린 시간을 측정하는 함수 구현

```kotlin
fun <T> loggingStopWatch(function: () -> T): T {
    val startAt = LocalDateTime.now()
    logger.info("Start At : $startAt")

    val result = function.invoke()

    val endAt = LocalDateTime.now()

    logger.info("End At : $endAt")
    logger.info("Logic Duration : ${Duration.between(startAt, endAt).toMillis()}ms")

    return result
}
```

**적용**

적용할 곳에 관련 함수를 인자로 전달해서 사용할 수 있다.

```kotlin
// Controller
@PostMapping("/members")
fun signIn() {
    loggingStopWatch { memberService.signIn() }
}
```

**단점**

다만, 단점이라고 생각이 드는 부분은 포인트컷을 사용하지 않기 때문에 전역적으로 사용하거나 적용 범위가 클 경우에는 번거로울 것으로 예상된다.

```kotlin
// Controller
@PostMapping("/members")
fun signIn() {
    a { 
        b {
            c {
                // logic
            }
        }
     }
}
```

또한, 해당 방법으로 여러 AOP를 적용하려고 할 경우 depth가 적용 횟수만큼 늘어나게 된다.

## 내부 함수 호출 극복

AOP를 사용하지 않아 프록시를 안 거치기 때문에 내부 함수 호출로 로직 적용이 가능해졌고 private, protected의 접근 제한자도 적용할 수 있다.

```kotlin
// Controller
@PostMapping("/members")
fun signIn() {
    this.signIn()
}

fun singIn(member: Member) = loggingStopWatch {
    memberRepository.save(member)
}
```

## @Transactional 극복

내부 함수 호출이 AOP 적용되기를 원할 때는 `@Transactional`를 사용할 때이다. `@Transactional`는 Spring Context에 엮어있는 트랜잭션 로직을 분석하여 구현하기에 무리가 있어 `@Transactional`를 그대로 사용하면서 적용하는 방법을 고안.

트랜잭션이 적용되게 하기 위해 `TxAdivce` 라는 Bean을 생성하여 trailing Lambdas 문법을 사용할 수 있는 함수를 구현하였다.

```kotlin
@Component
class TxAdvice {

    @Transactional
    fun <T> run(function: () -> T): T {
        contract {
            callsInPlace(function, kotlin.contracts.InvocationKind.EXACTLY_ONCE)
        }
        return function.run()
    }
}
```

하지만, 이런 식으로 사용했을 경우 자가 호출을 사용하기 위해서 매번 빈을 주입해야 하는 번거로움이 생긴다.

```kotlin
@Component
class Tx(
    _txAdvice: TxAdvice,
) {

    init {
        txAdvice = _txAdvice
    }

    companion object {
        private lateinit var txAdvice: TxAdvice

        fun <T> run(function: () -> T): T {
            return txAdvice.run(function)
        }
    }

    @Component
    class TxAdvice {

        @Transactional(propagation = Propagation.REQUIRES_NEW)
        fun <T> run(function: () -> T): T {
            return function()
        }
    }
}
```

위에처럼 bean을 한 번 더 감싸고 전역 메서드로 동일하게 trailing Lambdas 문법의 함수로 내부 `TxAdvice`의 `run` 함수를 호출해 줄 경우 `TxAdvice` Bean의 의존 없이 전역 메서드를 통해 전역적으로 트랜잭션을 사용할 수 있게 한다.

```kotlin
@Transactional
fun signIn() {
    signInInner()
}

fun signInInner() = Tx.run {
    val member = Member()
    memberRepository.save(member)
}
```

이제 내부 함수에서도 트랜잭션을 적용할 수 있게 되었다.

다음과 같이 사용했을 때 내부 함수를 대안으로 별도의 클래스를 만들어 메서드를 만들어 호출하는 방식이 있는데 계층 또는 depth가 깊어지기 때문에 복잡해져 유지보수에 어려움을 겪을 수 있는데 내부 함수 호출에 적용이 가능해지므로서 앞에서 말한 문제는 걱정하지 않아도 된다.

***@Cacheable 극복하기, Reverse Argument 등 더 자세한 내용 또는 추가 팁은 테크 블로그***