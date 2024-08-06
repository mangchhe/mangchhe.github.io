---
layout: post
title: Kotlin JSON Field Masking with Jackson
categories: spring
tags: spring
---

```kotlin
enum class MaskType {
    NAME, CONTACT
}

@Target(AnnotationTarget.FIELD)
@Retention(AnnotationRetention.RUNTIME)
annotation class JsonMask(val type: MaskType)

data class UserInfo(
    @JsonMask(MaskType.NAME)
    val name: String,
    @JsonMask(MaskType.CONTACT)
    val phoneNumber: String
) 
```

```kotlin
class NameMaskingSerializer : JsonSerializer<Any>() {
    override fun serialize(value: Any?, gen: JsonGenerator, serializers: SerializerProvider) {
        val maskedValue = (value as? String)?.let {
            if (it.length > 1) it.first() + "*".repeat(it.length - 1) else it
        } ?: ""
        gen.writeString(maskedValue)
    }
}

class ContactMaskingSerializer : JsonSerializer<Any>() {
    override fun serialize(value: Any?, gen: JsonGenerator, serializers: SerializerProvider) {
        val maskedValue = (value as? String)?.let {
            it.take(it.length - 4) + "*".repeat(4)
        } ?: ""
        gen.writeString(maskedValue)
    }
}

class MaskingModule : SimpleModule() {
    init {
        setSerializerModifier(object : BeanSerializerModifier() {
            override fun changeProperties(
                config: com.fasterxml.jackson.databind.SerializationConfig?,
                beanDesc: com.fasterxml.jackson.databind.BeanDescription?,
                beanProperties: MutableList<BeanPropertyWriter>
            ): MutableList<BeanPropertyWriter> {
                beanProperties.forEach { writer ->
                    val field: Field? = when (val member = writer.member) {
                        is AnnotatedField -> member.annotated
                        is AnnotatedMethod -> member.declaringClass.getDeclaredField(writer.name)
                        else -> null
                    }
                    field?.getAnnotation(JsonMask::class.java)?.let { annotation ->
                        val serializer: JsonSerializer<Any> = when (annotation.type) {
                            MaskType.NAME -> NameMaskingSerializer()
                            MaskType.CONTACT -> ContactMaskingSerializer()
                        }
                        writer.assignSerializer(serializer)
                    }
                }
                return beanProperties
            }
        })
    }
}
```

```kotlin
fun main() {
    val objectMapper = ObjectMapper().apply {
        registerKotlinModule()
        registerModule(MaskingModule())
    }

    val userInfo = UserInfo(
        name = "홍길동",
        "010-1234-5678",
    )

    val result = objectMapper.writeValueAsString(userInfo)
    println(result) // {"name":"홍**","phoneNumber":"010-1234-****"}
}
```