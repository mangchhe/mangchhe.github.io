---
layout: post
title: TimeLimiter 'name' recorded a timeout exception. - Resilience4j
categories: spring
tags: spring
---

> TimeLimiter 'name' recorded a timeout exception. 

`TimeLimiter` 가 timeout exception을 기록했다는 에러 코드가 발생하여 관련 내용을 찾아보았다.

```java
// TimeLieiter: 84
static TimeoutException createdTimeoutExceptionWithName(String name, @Nullable Throwable t) {
    TimeoutException timeoutException = new TimeoutException(String.format("TimeLimiter '%s' recorded a timeout exception.", name));
    if (t != null) {
        timeoutException.setStackTrace(t.getStackTrace());
    }

    return timeoutException;
}

// TimeLimiterImpl:49
public <T, F extends Future<T>> Callable<T> decorateFutureSupplier(Supplier<F> futureSupplier) {
        return () -> {
            Future<T> future = (Future)futureSupplier.get();

            try {
                T result = future.get(this.getTimeLimiterConfig().getTimeoutDuration().toMillis(), TimeUnit.MILLISECONDS);
                this.onSuccess();
                return result;
            } catch (TimeoutException var5) {
                TimeoutException e = var5;
                TimeoutException timeoutException = TimeLimiter.createdTimeoutExceptionWithName(this.name, e);
                this.onError(timeoutException);
                if (this.getTimeLimiterConfig().shouldCancelRunningFuture()) {
                    future.cancel(true);
                }

                throw timeoutException;
            }
        ...
        }
}
```

디버깅하여 코드를 따라가다 보니 해당 코드에서 데코레이터 패턴으로 감싸진 부분에서 실제 코드를 실행하는 부분에서 발생하는 것을 확인했다.

```java
T result = future.get(this.getTimeLimiterConfig().getTimeoutDuration().toMillis(), TimeUnit.MILLISECONDS);
```

> [Git](https://github.com/resilience4j/resilience4j?tab=readme-ov-file#4-resilience-patterns) & [Docs](https://resilience4j.readme.io/docs/timeout)
>
> - limits duration of execution
> 
> - the timeout duration
> 
> - whether cancel should be called on the running future

문서에 내용을 종합해 보면 `Timelimiter`의 설정은 실행 시간을 제한하는 timeout 설정이고, 실행 중인 `future`에 취소를 요청해야 하는지를 결정하는 부분이다.

그러면 위 코드가 이해가 간다. 실행 중인 `future` 가 `TimeLimiterConfig`에 설정된 duration(timeout) 시간을 초과하게 될 경우 `TimeoutException` 이 발생하게 된다.

`TimeLimiterConfig`의 설정을 찾아오는 부분을 좀 더 디테일하게 확인해 보자.

`loadTimeLimiter` 함수에서 설정된 `TimeLimiterConfig`를 가져와서 설정하는 것을 볼 수 있고 값이 없으면 default 값을 사용하는 것을 알 수 있다.

default 값은 1s 로 잡혀있어서 헤비한 api일 경우 조금 타이트할 수 있다.

```java
// Resilience4JCircuitBreaker
// :62
public <T> T run(Supplier<T> toRun, Function<Throwable, T> fallback) {
    Map<String, String> tags = Map.of("group", this.groupName);
    Optional<TimeLimiter> timeLimiter = this.loadTimeLimiter();

    ...

    restrictedCall = (Callable)timeLimiter.map((tl) -> {
                return TimeLimiter.decorateFutureSupplier(tl, decorator);
    }
    decorator = io.github.resilience4j.circuitbreaker.CircuitBreaker.decorateSupplier(defaultCircuitBreaker, toRun);
}

// :132
private Optional<TimeLimiter> loadTimeLimiter() {
    return this.disableTimeLimiter ? Optional.empty() : Optional.of((TimeLimiter)this.timeLimiterRegistry.find(this.id).orElseGet(() -> {
        return (TimeLimiter)this.timeLimiterRegistry.find(this.groupName).orElseGet(() -> {
            return this.timeLimiterRegistry.timeLimiter(this.id, this.timeLimiterConfig, this.tags);
        });
    }));
}

// TimeLimiterConfig
public class TimeLimiterConfig implements Serializable {
    private static final long serialVersionUID = 2203981592465761602L;
    private static final String TIMEOUT_DURATION_MUST_NOT_BE_NULL = "TimeoutDuration must not be null";
    private Duration timeoutDuration = Duration.ofSeconds(1L);
    private boolean cancelRunningFuture = true;

    private TimeLimiterConfig() {
    }
    ...
}
```

결국 문제의 원인은 timeout 설정에 있었고, 나는 default 값을 변경하여 해결하였다.

기본적으로 스프링 스타터 팩에서 제공하는 `resilience4j`는 사용하고 있을 것이고 추가로 `Timelimiter` 설정을 하기 위해서는 `resilience4j-core` 또는 `resilience4j-timelimiter` 의존성을 추가하면 된다.

```gradle
implementation("org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j:3.1.2")
implementation("io.github.resilience4j:resilience4j-core")
implementation("io.github.resilience4j:resilience4j-timelimiter")
```

의존성을 추가한 후, `Timelimiter`의 default 값을 수정하게 되면 정상 동작하는 것을 확인할 수 있다.

```yml
resilience4j:
  timelimiter:
    configs:
      default:
        timeoutDuration: 10s
        cancelRunningFuture: true
```