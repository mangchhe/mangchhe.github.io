---
title: "[Kotlin] Null 처리 방법 - ? ?. ?: !! as?"
description: 코틀린에서 Null을 처리하는 다양한 방법
categories:
 - Kotlin
tags:
 - Kotlin
 - "null"
---

# 개요

<hr>

- 기본적으로 자바에서 변수는 null을 항상 허용하지만 코틀린에서는 그렇지 않다.
- 코틀린에서 null을 처리할 수 있는 방법에 대해서 알아보자.
- 예제 소스 : [바로가기](https://github.com/mangchhe/kotlin-study/tree/main/src/4_null_safety)

# 널이 가능한 타입 (?)

<hr>

- Kotlin에서 null이 가능한 타입으로 만들어주기 위해서는 타입 뒤에 ?를 붙여서 명시해주어야 한다.

```java
/* 변수 */
var a: String = "abc"
a = null // 불가능

var b: String? = "abc"
b = null // 가능

/* 컬렉션 */
val list: List<Int> = listOf(1, 2, null, 4) // 불가능

val nullableList: List<Int?> = listOf(1, 2, null, 4) // 가능
```

# 널 체크 (?.)

<hr>

- ?. 를 이용하게 되면 변수/함수가 null이 아닐 경우에만 가져오거나 호출할 수 있다.
- null일 경우에는 null 반환한다.

```java
// 조건문을 이용한 방식
var a: String? = "abs"
var b: Int?
if (a != null) {
    b = a.length
}
// 다른 방식
val c: String? = null
var d: Int?
d = c?.length

// let과 함께 쓰면 null이 아닌 값만 추출 가능
val listWithNulls: List<String?> = listOf("Kotlin", null)
for (item in listWithNulls) {
    item?.let { println(it) }
}
```

# elvis 연산자 (?:)

<hr>

- ?.과 ?: 연산자를 함께 쓰면 null일 경우에는 사용자가 정의한 값을 지정할 수 있다.
- 값 이외에도 return, throw 같은 형태로도 사용할 수 있다.

```java
fun foo(node: Node): String? {
    val parent = node.parentNode ?: return null
    val name = node.nodeName ?: throw IllegalArgumentException("name expected")
    return ""
}

fun main() {
    var a: String? = null
    val l: Int = if (a != null) a.length else -1
    val ll: Int = a?.length ?: -1
}
```

# Not Null 보증 (!!)

<hr>

- 널이 아님을 보증할 때 사용한다.
- 만약 null일 경우 NPE 발생시킨다.

```java
fun main() {
    var a: String? = "Kotlin"
    val l = a!!.length
}
```

# 안전 캐스팅 (as?)

<hr>

- as로 지정한 타입으로 캐스팅이 불가능하다면 null을 반환한다.
- 만약 null일 경우 NPE 발생시킨다.

```java
fun main() {
    var a: String? = null

    var aInt: Int? = a as? Int

    var bInt: Int? = a as Int // NPE 발생
}
```