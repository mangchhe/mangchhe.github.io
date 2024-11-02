---
layout: post
title: Configuring Multiple Firebase Beans in Spring Boot
categories: spring
tags: spring
---

> - [https://zuminternet.github.io/FCM-PUSH/](https://zuminternet.github.io/FCM-PUSH/)
> 
> - [https://www.baeldung.com/spring-fcm](https://www.baeldung.com/spring-fcm)

```kotlin
@Component
class FirebaseAppProvider(
    private val firebaseProperties: FirebaseProperties
) {
    private val firebaseApps = ConcurrentHashMap<String, FirebaseApp>()

    @PostConstruct
    fun initializeFirebaseApps() {
        val directory = ResourceUtils.getFile(firebaseProperties.configPath)
        require(directory.isDirectory) { "Config path must be a directory" }

        directory.listFiles { file -> file.name.endsWith(".json") }
            ?.forEach { file ->
                val projectName = file.nameWithoutExtension

                val options = FirebaseOptions.builder()
                    .setCredentials(GoogleCredentials.fromStream(FileInputStream(file)))
                    .build()

                FirebaseApp.initializeApp(options, projectName).also { app ->
                    firebaseApps[projectName] = app
                }
            }
    }

    fun getFirebaseApp(projectName: String): FirebaseApp {
        return firebaseApps[projectName] ?: throw IllegalArgumentException(
            "Firebase app not found for project: $projectName"
        )
    }
}

fun main() {
    val firebaseApp = firebaseAppProvider.getFirebaseApp(projectName)
    val response = FirebaseMessaging.getInstance(firebaseApp).send(Message())
}
```