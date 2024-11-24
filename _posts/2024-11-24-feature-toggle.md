---
layout: post
title: Feature Toggle (aka Feature Flag)
categories: cs
tags: cs
---

> [https://medium.com/daangn/%EB%A7%A4%EC%9D%BC-%EB%B0%B0%ED%8F%AC%ED%95%98%EB%8A%94-%ED%8C%80%EC%9D%B4-%EB%90%98%EB%8A%94-%EC%97%AC%EC%A0%95-2-feature-toggle-%ED%99%9C%EC%9A%A9%ED%95%98%EA%B8%B0-b52c4a1810cd](https://medium.com/daangn/%EB%A7%A4%EC%9D%BC-%EB%B0%B0%ED%8F%AC%ED%95%98%EB%8A%94-%ED%8C%80%EC%9D%B4-%EB%90%98%EB%8A%94-%EC%97%AC%EC%A0%95-2-feature-toggle-%ED%99%9C%EC%9A%A9%ED%95%98%EA%B8%B0-b52c4a1810cd)

코드를 수정하지 않고 시스템의 동작을 바꿀 수 있는 기술

사용자에게 새로운 기능을 빠르고 안전하게 제공하는 데 도움을 주는 패턴

##### 구성 요소

- 토큰 포인트 (Toggle Point) : 새로운 기능과 기존 기능을 구분하는 코드 지점
- 토글 라우터 (Toggle Router) : Feature가 활성화되었는지 판단하는 로직
- 토글 설정 (Toggle Configuration) : Feature의 ON/OFF 상태를 결정하는 상태 값. 다양한 저장소에서 관리

##### 설계 원칙

- 기본 동작
  - 켜지면 새로운 기능 활성화, 꺼지면 기능 동작 유지
- 토글 포인트와 라우터 분리
  - 구분 로직을 한 곳에 집중하여 유지보수성을 높임
- 특정 저장소 의존 제거
  - 설정값은 다양한 저장소에서 관리 가능
- 코드 배포 없이 동적 설정 지원
  - 설정값을 주기적으로 동기화해 실시간 변경 지원
- 사용하지 않는 토글 제거
  - 유지보수 비용 감소 및 코드 간소화

##### 구현

Annotation 기반 토글 라우터
- `@FeatureToggle`과 `@FeatureToggleFallback`을 사용해 신규 및 기존 동작을 구분
- AOP를 활용해 깔끔한 코드로 동작 분리

```kotlin
// 설정값 관리
interface FeatureToggleProvider {
    fun isEnabled(key: String): Boolean
}

class FeatureToggleProviderImpl(private val repository: ToggleRepository) : FeatureToggleProvider {
    private val toggleConfigMap: AtomicReference<Map<String, ToggleConfiguration>> = AtomicReference(emptyMap())

    @Scheduled(fixedDelay = 5000)
    fun syncToggleConfiguration() {
        val newToggleConfigMap = repository.findAll()
            .associateBy({ it.key }, { this.parseConfig(it) })
        toggleConfigMap.set(newToggleConfigMap)
    }
    ...
}

// old, new feature 적용 로직
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class FeatureToggle(
    val key: String,
    val fallbackMethod: String = "",
)

@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class FeatureToggleFallback

@Aspect
@Component
class FeatureToggleAspect(private val featureToggleProvider: FeatureToggleProvider) {
    private val fallbackExecutor = FeatureToggleFallbackExecutor()
    @Pointcut(value = "@within(featureToggle) || @annotation(featureToggle)", argNames = "featureToggle")
    fun matchAnnotatedClassOrMethod(featureToggle: FeatureToggle) {
    }
    @Around(value = "matchAnnotatedClassOrMethod(featureToggleAnnotation)", argNames = "proceedingJoinPoint, featureToggleAnnotation")
    fun featureToggleAroundAdvice(proceedingJoinPoint: ProceedingJoinPoint, featureToggleAnnotation: FeatureToggle): Any {
        ...
        // Toggle Point
        if (featureToggleProvider.isEnabled(key)) {
            return proceedingJoinPoint.proceed()
        }
        return fallbackExecutor.execute(proceedingJoinPoint, key, fallbackMethod)
    }
}

class MyFeatureProcessor {
    @FeatureToggle(key = "NEW_COOL_FEATURE", fallbackMethod = "oldFeature")
    fun newCoolFeature(): String {
        return "NewCoolFeature"
    }

    @FeatureToggleFallback
    fun oldFeature(): String {
        return "OldFeature"
    }
}
```

##### 활용 사례

- 새로운 기능 배포 전에 팀 내부 테스트 → Permissioning Toggle
- 위험성이 높은 기능 → Canary Toggle로 점진적 배포
- 배포 독립성 보장(A 서버와 B 서버 간 종속성 해결) → Feature Toggle 사용