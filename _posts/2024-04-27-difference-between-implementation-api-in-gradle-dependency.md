---
layout: post
title: Difference between implementation, api in Gradle Dependency
categories: spring
tags: spring
---

> **implementation**
> 
> The dependencies required to compile the production source of the project **which are not part of the API exposed by the project.** For example the project uses Hibernate for its internal persistence layer implementation.

**프로젝트에 노출되는 API의 일부분이 아닌** 프로젝트의 프로덕션 소스를 컴파일하는 데 필요한 종속성이다.

<p align="center">
    <img src='/assets/postImages/DifferenceBetweenImplementationApiInGradleDependency/implementation.png'>
</p>

- `compileClassPath` : 프로젝트 컴파일에 필요한 모든 클래스와 라이브러리 경로를 지정하는 역할
- `runtimeClassPath` : 런타임 때 필요한 모든 클래스와 라이브러리 경로를 지정하는 역할

> **api**
>
> The dependencies required to compile the production source of the project **which are part of the API exposed by the project.** For example the project uses Guava and exposes public interfaces with Guava classes in their method signatures.

**프로젝트에 노출되는 API의 일부분인** 프로젝트의 프로덕션 소스를 컴파일하는 데 필요한 종속성이다.

<p align="center">
    <img src='/assets/postImages/DifferenceBetweenImplementationApiInGradleDependency/api.png' width="650">
</p>

`api`도 `implementation`이랑 동일하게 컴파일, 런타임의 클래스패스 설정이 모두 적용된다.

### implementation, api 차이점

앞서 말한 대로 둘 다 컴파일, 런타임 클래스패스가 설정된다는 부분에서 동일하지만, 전이 의존성(transitive dependency)에서 차이점이 나타난다.

```kotlin
// A module
dependencies {
    api("1 library")
    implementation("2 library")
}

// B module
dependencies {
    implementation(project(":a"))
}
```

위와 같이 설정한 후 Gradle 설정을 보게 되었을 때 `implementation`은 컴파일 경로 설정을 하지 않는 것을 알 수 있다.

- implementation
  - `runtimeClassPath`
  - `testRuntimeClassPath`
- api
  - `compileClassPath`
  - `runtimeClassPath`
  - `testCompileClassPath`
  - `testRuntimeClassPath` 

<br>
이로써 알 수 있는 점은 `implementation`은 `compileClassPath`가 추가되지 않기 때문에 모듈 B에서 2 library의 API를 직접적으로 사용할 수 없다. 하지만 `runtimeClassPath`은 추가되어 있기 때문에 런타임에서 사용하는 것은 가능하다.

그리고 만약 `api`로 의존성 설정했는데 해당 라이브러리 버전이 변경되었을 경우 속한 모듈은 API를 직접적으로 사용할 수 있기 때문에 종속된 모듈이 모두 재빌드가 되어야 한다. 하지만 반대로 `implementation`은 직접적으로 API를 사용할 수 없기 때문에 속한 모듈 모두 재빌드가 필요 없고 의존하는 모듈만 재빌드하면 된다.

### References

- [dependency_management_for_java_projects](https://docs.gradle.org/current/userguide/dependency_management_for_java_projects.html#sec:configurations_java_tutorial)
- [implementation image](https://docs.gradle.org/current/userguide/java_plugin.html#tab:configurations)
- [api image](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_configurations_graph)