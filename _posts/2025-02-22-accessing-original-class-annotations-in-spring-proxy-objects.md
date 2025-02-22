---
layout: post
title: Accessing Original Class Annotations in Spring Proxy Objects
categories: spring
tags: spring
---

Spring에서 프록시 객체는 AOP, @Transactional, @Cacheable 등 다양한 기능을 구현하기 위해 사용된다. 이러한 프록시 객체는 원본 클래스의 어노테이션 정보를 직접적으로 가져올 수 없는 제약이 있다.

이 문제를 해결하기 위해 프록시 객체에서 원본 객체를 찾아내는 여러 가지 방법이 존재한다.

1.

Spring AOP에서 제공하는 유틸리티를 사용하여 프록시의 실제 타겟 클래스를 얻을 수 있다.

```kotlin
val targetClass = AopProxyUtils.ultimateTargetClass(proxyService)
```

2.

프록시 객체가 구현하는 Advised 인터페이스를 통해 실제 타겟 객체를 직접 참조할 수 있다.

```kotlin
val targetObject = getTargetObject(proxyService)

fun getTargetObject(proxy: Any): Any? {
    return if (AopUtils.isAopProxy(proxy) && proxy is Advised) {
        proxy.targetSource.target
    } else {
        proxy
    }
}
```

3.

CGLIB를 사용한 프록시의 경우, 원본 클래스는 프록시 클래스의 슈퍼클래스이다. 클래스 이름에 "CGLIB"가 포함되어 있는지 확인하여 원본 클래스를 찾을 수 있다.

```kotlin
val originalClass = getOriginalClass(proxyService)

fun getOriginalClass(obj: Any): Class<*> {
    return if (obj::class.java.name.contains("CGLIB")) {
        obj::class.java.superclass
    } else {
        obj::class.java
    }
}
```