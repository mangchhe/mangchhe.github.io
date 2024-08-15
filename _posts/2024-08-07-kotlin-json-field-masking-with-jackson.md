---
layout: post
title: Kotlin JSON Field Masking with Jackson
categories: spring
tags: spring
---

> [개발자 대상 개인정보 보호조치 적용 안내서](https://www.kisa.or.kr/2060301/form?postSeq=10&page=1)

![masking-rules](/assets/postImages/KotlinJsonFieldMaskingWithJackson/masking-rules.png)

**Enum and Annotation**

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

**Custom Serializers**

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
```

**Serializer Module**

> [SimpleModule](https://fasterxml.github.io/jackson-databind/javadoc/2.4/com/fasterxml/jackson/databind/module/SimpleModule.html)
>
> Jackson의 Module 클래스를 상속하여 만들어진 클래스이며 해당 모듈은 커스텀한 serializers와 deserializers을 등록할 수 있게 한다.
>
> [BeanSerializerModifier](https://fasterxml.github.io/jackson-databind/javadoc/2.5/com/fasterxml/jackson/databind/ser/BeanSerializerModifier.html)
>
> Jackson에서 제공하는 추상 클래스이며 Bean의 직렬화 과정에서 커스텀할 수 있는 API를 정의한다.

```kotlin
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

**Test**

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