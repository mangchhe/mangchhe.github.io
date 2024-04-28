---
layout: post
title: Spring Boot Multi Module with Kotlin
categories: spring
tags: spring
---

### 초기 세팅

#### gradle.properties (root)

```kotlin
applicationVersion = 0.0.1
projectGroup = me.hajoo

springBootVersion = 3.2.5
springDependencyManagementVersion = 1.1.4

kotlinVersion = 1.9.23
```

#### build.gradle.kts (root)

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
