---
layout: post
title: Spring Boot Multi Module with Kotlin
categories: spring
tags: spring
---

### 초기 세팅

#### gradle.properties (root)

프로젝트의 전역 속성을 설정한다. 프로젝트 버전, 그룹 ID 및 사용되는 주요 라이브러리의 버전을 작성하여 관리를 수월하게 한다.

```kotlin
applicationVersion = 0.0.1
projectGroup = me.hajoo

springBootVersion = 3.2.5
springDependencyManagementVersion = 1.1.4

kotlinVersion = 1.9.23
```

#### build.gradle.kts (root)

프로젝트의 빌드 과정을 구성한다. Kotlin과 Spring Boot 플러그인을 포함하여 필요한 플러그인을 적용하고, 모든 프로젝트와 하위 프로젝트의 공통 설정을 정의한다.

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
    id("org.springframework.boot") apply false
    id("io.spring.dependency-management") apply false
    kotlin("jvm")
    kotlin("plugin.spring") apply false
}

val projectGroup: String by project
val applicationVersion: String by project

allprojects {
    group = projectGroup
    version = applicationVersion

    repositories {
        mavenCentral()
    }
}

subprojects {
    apply(plugin = "org.springframework.boot")
    apply(plugin = "io.spring.dependency-management")
    apply(plugin = "org.jetbrains.kotlin.plugin.spring")
    apply(plugin = "kotlin")
    apply(plugin = "kotlin-kapt")

    dependencies {
        implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
        implementation("org.jetbrains.kotlin:kotlin-reflect")
        testImplementation("org.springframework.boot:spring-boot-starter-test")
    }

    tasks.withType<KotlinCompile> {
        kotlinOptions {
            freeCompilerArgs += "-Xjsr305=strict"
            jvmTarget = "21"
        }
    }

    tasks.withType<Test> {
        useJUnitPlatform()
    }
}
```

#### settings.gradle.kts (root)

프로젝트 플러그인 버전 관리를 통합적으로 적용한다.

```kotlin
rootProject.name = "multi-module"

pluginManagement {
    val kotlinVersion: String by settings
    val springBootVersion: String by settings
    val springDependencyManagementVersion: String by settings

    resolutionStrategy {
        eachPlugin {
            when (requested.id.id) {
                "org.jetbrains.kotlin.jvm" -> useVersion(kotlinVersion)
                "org.jetbrains.kotlin.plugin.spring" -> useVersion(kotlinVersion)
                "org.springframework.boot" -> useVersion(springBootVersion)
                "io.spring.dependency-management" -> useVersion(springDependencyManagementVersion) }
        }
    }
}
```

### 모듈 설정

#### settings.gradle.kts

필요한 모듈을 포함시켜 Gradle이 프로젝트 구조를 인식할 수 있도록 한다.

```kotlin
rootProject.name = "multi-module"

...

include(
    "hj-app-api"
)

include(
    "hj-common",
    "hj-core"
)
```

#### build.gradle.kts

실행 가능한 애플리케이션의 경우 `bootJar`를 true로 하고 그렇지 않은 경우 `jar`를 true로 한다.

##### 애플리케이션 (api)

```kotlin
tasks.bootJar { enabled = true }
tasks.jar { enabled = false }

dependencies {
    // 필요한 의존성 설정
    implementation(project(":hj-common"))
    implementation(project(":hj-core"))

    implementation("org.springframework.boot:spring-boot-starter-web")
}
```

##### 라이브러리 (common, core, client)

```kotlin
tasks.bootJar { enabled = false }
tasks.jar { enabled = true }

dependencies {
  // ...
}
```
