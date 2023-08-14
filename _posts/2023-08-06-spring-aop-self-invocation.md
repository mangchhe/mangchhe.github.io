---
layout: post
title: Self-invocation does not work for proxies in Spring AOP
categories: spring
tags: spring
---

```kotlin
class Test {
    
    @Transactional
    fun a() {
        b()
    }
    
    fun b() {
        // logic
    }
}
```

위 코드에서 a 메서드에서 같은 클래스에 있는 b 메서드를 호출하고 있다. 현재 b 메서드는 a 트랜잭션 내에 감싸져 있기 때문에 둘은 같은 트랜잭션에서 실행되고 있고 문제가 없다.

```kotlin
class Test {
    
    fun a() {
        b()
    }
    
    @Transactinoal
    fun b() {
        // logic
    }
}
```

하지만 다음과 같이 b 메서드에 트랜잭션이 걸려있고 자가 호출을 하게 될 경우에는 얘기가 다르다. 해당 경우에는 트랜잭션의 범위가 b 메서드의 시작과 끝인데 a에서 b 메서드를 호출할 때 프록시로 호출하는 것이 아닌 plain object 자체를 호출하기 때문에 트랜잭션이 적용되지 않아 의도한 결과대로 작동하지 않는다.

```kotlin
class Test {
    
    @Transactional
    fun a() {
        b()
        println(TransactionSynchronizationManager.getCurrentTransactionName())
    }
    
    @Transactinoal(propagation = Propagation.REQUIRES_NEW)
    fun b() {
        println(TransactionSynchronizationManager.getCurrentTransactionName())
    }
}
```

한 가지 더 예를 들면 전파 단계를 REQUIRES_NEW로 설정했을 경우 b 메서드에서는 새로운 트랜잭션이 시작될 것을 기대할 수 있다. 하지만, 프록시로 호출하는 것이 아니기 때문에 해당 전파 단계는 적용되지 않고 동일한 트랜잭션(부모 트랜잭션)이게 된다.

>
[Understanding AOP Proxies](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html#aop-understanding-aop-proxies)
>
self-invocation is not going to result in the advice associated with a method invocation getting a chance to run.
The best approach (the term "best" is used loosely here) is to refactor your code such that the self-invocation does not happen.

자가 호출은 해당 메서드 호출과 관련된 advice가 실행될 기회를 얻지 못할 것이다. 그러므로 자가 호출을 사용하지 않는 방식으로 리팩토링하는 것이 최선의 접근 방법이라고 한다.

위에서 예를 들었다시피 `@Transactional`과 같은 DB 트랜잭션 관리는 스프링 AOP를 사용하고 있기 때문에 마찬가지로 자가 호출의 경우 트랜잭션에 영향을 줄 수 없다.

>
[Aspect Oriented Programming with Spring](https://docs.spring.io/spring-framework/docs/2.5.5/reference/aop.html)
>
Due to the proxy-based nature of Spring's AOP framework, protected methods are by definition not intercepted, neither for JDK proxies nor for CGLIB proxies

추가로 같은 클래스가 아니더라도 주의해야 할 사항이 있다. 메서드는 public method에서 밖에 사용하지 못한다. AOP는 프록시 기반으로 protected method는 JDK, CGLIB proxy에서 intercept 하지 않는다고 나와 있다. private은 override와 상속이 불가능하기 때문에 당연히 적용할 수 없다.