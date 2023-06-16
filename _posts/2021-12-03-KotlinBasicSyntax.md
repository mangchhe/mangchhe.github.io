---
title: "Kotlin basic"
categories: kotlin
tags: kotlin
---

# 시작(entry point)

<hr>

코틀린 어플리케이션은 main 함수로부터 시작한다. 시작은 역시나 Hello World!

``` java
fun main() {
    println("Hello World!")
}
```

해당 소스는 어플리케이션을 동작시킬 때 명령줄에 인자를 넣어 값을 전달할 수 있다.

``` java
fun main(args: Array<String>) {
    println(args.contentToString())
}
```

# 표준 출력(stardard output)

<hr>

자바에서 System.out만 없어진 형태이고 오버로딩을 이용해 다양한 타입으로 인자를 받는 것을 알 수 있다.

``` java
fun main() {
    print("Hello ")
    print("World!\n")
    println(42)
}
```

# 함수(function)

<hr>

## 함수 구현

함수 원형은 `fun 함수이름(매개변수): 리턴타입 {본문}`으로 구현된다.

``` java
fun add(a: Int, b: Int): Int {
    return a + b
}
```

## 다른 함수 표현식

`fun 함수이름(매개변수) = 본문` 로도 표현되며 리턴타입은 추론되어 반환된다.

``` java
fun subtract(a: Int, b: Int) = a - b
```

## 리턴타입 Unit

두 함수는 리턴타입이 Unit(자바로 치면 void)인데 명시적으로 적어주어도 되고 생략해도 상관이 없다.

``` java
fun printAdd(a: Int, b: Int): Unit {
    println("add of $a and $b is ${a + b}")
}

fun printSubtract(a: Int, b: Int) {
    println("subtract of $a and $b is ${a - b}")
}
```

## 결과

``` java
fun main() {
    println(add(1, 2))
    println(subtract(2, 1))
    printAdd(1, 2)
    printSubtract(2, 1)
}
/* 결과
3
1
add of 1 and 2 is 3
subtract of 2 and 1 is 1
*/
```

# 변수(Variable)

<hr>

## 지역변수

``` java
fun main() {
    var a: Int = 1
    var b = 2
    var c: Int
    c = 3
}
```

## 전역변수

``` java
var x = 0

fun incrementX() {
    x += 1
}

fun main() {
    incrementX()
    print(x)
}
/* 결과
1
*/
```

# 클래스 & 인스턴스

<hr>

코틀린에서는 클래스와 메소드가 기본적으로 final 속성을 가지기 때문에 상속을 하거나 오버라이딩을 허용하고 싶다면 open 키워드를 사용해야한다.

``` java
open class Shape

class Rectangle(var height: Double, var length: Double): Shape() {
    var perimeter = (height + length) * 2
}

fun main() {
    var rectangle = Rectangle(5.0, 2.0)
    println("The perimeter is ${rectangle.perimeter}")
}
```

# String Template

<hr>

문자열에 변수를 포함시키기 위해 사용하기 위해 쉽게 표현하는 기능, 문자열 안에 `$변수`, `${작업}`를 이용해서 표현한다. 

``` java
var a = 1
var s1 = "a is $a"

a = 2
var s2 = "${s1.replace("is", "was")}, but now is $a"

println(s1)
println(s2)
/* 결과
a is 1
a was 1, but now is 2
*/
```

# 조건문

<hr>

``` java
fun maxOf(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}

fun maxOf2(a: Int, b: Int) = if (a > b) a else b

fun main() {
    println(maxOf(5, 4))
    println(maxOf2(5, 4))
}
/* 결과
5
5
*/
```

# 반복문

<hr>

``` java
fun main() {
    var items = listOf("apple", "banana", "kiwifruit")
    for (item in items) {
        println(item)
    }
    for (index in items.indices) {
        println("item at $index is ${items[index]}")
    }
    var index = 0
    while (index < items.size) {
        println("item at $index is ${items[index]}")
        index++
    }
}
```

# when

<hr>

when은 java의 switch문과 비슷하다는 느낌을 받았다.

``` java
fun describe(obj: Any): String =
    when (obj) {
        1 -> "One"
        "Hello" -> "Greeting"
        is Long -> "Long"
        !is String -> "Not a string"
        else -> "Unknown"
    }

fun main() {
    println(describe(1))
    println(describe(2))
    println(describe("Hello"))
    println(describe(3L))
    println(describe("World"))
}
/* 결과
One
Not a string
Greeting
Long
Unknown
*/
```

# range

<hr>

파이썬의 range와 같이 iteration을 만들어주는 구문이라고 생각하면 될 것 같다. 여기서는 ``시작 인덱스..끝 인덱스``로 되어있고 마지막 인덱스 또한 포함한다는 것을 주의하면 될 것 같다.

솔직히 이 부분은 살짝 더 귀찮은 것 같은.. 본론으로 증가 크기를 변경하기 위해서는 step을 이용하고 반대로 내려가려고 할 경우 downTo를 뒤에 붙여 적절하게 사용하면 된다. 이때 step을 음수로 준다던가 10..1와 같이 반대로 하는 것은 안된다.

여기서 !in이 있는데 python에 not in과 같고 a !in b는 a가 b에 속하지 않는지 여부를 반환한다.

``` java
fun main() {
    var x = 10
    var y = 9
    if (x in 1..y + 1) {
        println("fits in range")
    }

    var list = listOf("a", "b", "c")

    if (-1 !in 0..list.lastIndex) {
        println("-1 is out of range")
    }
    if (list.size !in list.indices) {
        println("list size is out of valid list indices range, too")
    }

    for (x in 1..5) {
        print(x)
    }
    println()
    for (x in 1..10 step 2) {
        print(x)
    }
    println()
    for (x in 9 downTo 0 step 3) {
        print(x)
    }
}
/* 결과
fits in range
-1 is out of range
list size is out of valid list indices range, too
12345
13579
9630
*/
```

# collection

<hr>

listOf는 자바의 Arrays.asList()와 동일하다고 생각하면 되고 filter, map, forEach와 같이 익숙한 람다 표현식을 지원하는 것을 볼 수 있다.

``` java
fun main() {
    val items = listOf("apple", "banana", "kiwifruit")

    for (item in items) {
        println(item)
    }

    when {
        "orange" in items -> println("juicy")
        "apple" in items -> println("apple is fine too")
    }

    var fruits = listOf("banana", "avocado", "apple", "kiwifruit")
    fruits
        .filter { it.startsWith("a") }
        .sortedBy { it }
        .map { it.uppercase() }
        .forEach { println(it) }
}
/* 결과
apple
banana
kiwifruit
apple is fine too
APPLE
AVOCADO
*/
```

# nullable value & null check

<hr>

default로 null을 허용하지 않으며 허용하기 위해서는 자료형 뒤에 `?`를 붙여 nullable value를 만들어주게 된다. 자바에서 `@Nullable` `@NonNull` 이런 것보다 훨씬 간단한 거 같아서 마음에 든다. 

``` java
fun parseInt(str: String): Int? {
    // 문자열 값이 정수면 정수로 변환 후 반환 아니면 null
    return str.toIntOrNull(); 
}
fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    var y = parseInt(arg2)

    // null check
    if (x != null && y != null) {
        println(x * y)
    }
    else {
        println("$arg1 or $arg2 is not a number")
    }
}

fun main() {
    printProduct("99", "1")
    printProduct("a1", "12")
}
/* 결과
99
a1 or 12 is not a number
*/
```

# 타입체크 & 자동 변환

<hr>

is 연산자를 통해서 타입을 체크할 수 있고 이 연산자를 이용해 타입 체크가 된 인스턴스일 경우 명시적으로 타입 변환을 할 필요가 없다. 여기서 소스 본문에 있는 Any는 자바의 Object 모든 클래스의 최상위 조상 클래스라고 보면 된다. 자세한 내용은 주석 참고 

``` java
fun getStringLength(obj: Any): Int? {
    // if문에서 obj가 String임을 확인
    if (obj is String) {
        // String으로 자동 타입 변환
        return obj.length
    }
    return null
}

fun getStringLength2(obj: Any): Int? {
    // if문에서 obj가 String이 아님을 확인
    if (obj !is String) return null
    // Int로 자동 타입 변환
    return obj.length
}

fun getStringLength3(obj: Any): Int? {
    // if문에서 obj가 String이 아님을 확인
    if (obj is String && obj.length > 0) {
        // String으로 자동 타입 변환
        return obj.length
    }
    return null
}

fun main() {
    println(getStringLength("Hello World"))
    println(getStringLength(3))

    println(getStringLength2(""))
    println(getStringLength2(3))

    println(getStringLength3(""))
    println(getStringLength3(3))
}
/* 결과
11
null
0
null
null
null
*/
```

# Reference

<hr>

[https://kotlinlang.org/docs/basic-syntax.html#type-checks-and-automatic-casts](https://kotlinlang.org/docs/basic-syntax.html#type-checks-and-automatic-casts)