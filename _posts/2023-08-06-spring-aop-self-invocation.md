---
layout: post
title: Self-invocation does not work for proxies in Spring AOP
categories: spring
tags: spring
---

>
[Understanding AOP Proxies](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html#aop-understanding-aop-proxies)
>
self-invocation is not going to result in the advice associated with a method invocation getting a chance to run.
The best approach (the term "best" is used loosely here) is to refactor your code such that the self-invocation does not happen.

자가 호출은 해당 메서드 호출과 관련된 advice가 실행될 기회를 얻지 못할 것이다. 그러므로 자가 호출을 사용하지 않는 방식으로 리팩토링하는 것이 최선의 접근 방법이라고 한다.

예로 `@Transactional`과 같은 DB 트랜잭션 관리는 스프링 AOP를 사용하고 있기 때문에 마찬가지로 자가 호출에 트랜잭션이 영향을 줄 수 없다.

>
[Aspect Oriented Programming with Spring](https://docs.spring.io/spring-framework/docs/2.5.5/reference/aop.html)
>
Due to the proxy-based nature of Spring's AOP framework, protected methods are by definition not intercepted, neither for JDK proxies nor for CGLIB proxies

추가로 메서드는 public method에서 밖에 사용하지 못한다. AOP는 프록시 기반으로 protected method는 JDK, CGLIB proxy에서 intercept 하지 않는다고 나와 있다. private은 override와 상속이 불가능하기 때문에 당연히 적용할 수 없다.