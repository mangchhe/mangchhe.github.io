---
layout: post
title: Kotlin Regex
categories: kotlin
tags: kotlin
---

패턴에 일치하는 문자열을 찾거나 치환하거나 주변 문자열을 분리하기 위해서 `Regex` 를 사용한다.

## 정규 표현식 객체 생성 방법

[`kotlin.text.Regex`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-regex) 클래스를 사용하고 객체 생성에는 여러 방법이 있다.

#### 생성자

```kotlin
val regex = Regex("\\d+")

// kotlin.text.Regex 정의
public actual constructor(pattern: String) : this(Pattern.compile(pattern))
```

#### 확장 함수

```kotlin
val regex = "\\d+".toRegex()

// kotlin.text 정의
public inline fun String.toRegex(): Regex = Regex(this)
```

#### 정적 팩토리 메서드

```kotlin
val regex = Regex.fromLiteral("\\d+")

// kotlin.text 정의
public actual fun fromLiteral(literal: String): Regex = literal.toRegex(RegexOption.LITERAL)
```

## Regex 함수

#### 조회

- 패턴과 일치하는 문자열 탐색

```kotlin
fun find(input: CharSequence, startIndex: Int = 0): MatchResult?
```

- 패턴과 일치하는 문자열 모두 탐색

```kotlin
fun findAll(input: CharSequence, startIndex: Int = 0): Sequence<MatchResult>
```

- 패턴과 문자열이 완전히 일치하는 부분 탐색

```kotlin
fun matchEntire(input: CharSequence): MatchResult?
```

- 주어진 index에서 패턴과 문자열이 완전히 일치하는지 부분 탐색

```kotlin
fun matchAt(input: CharSequence, index: Int): MatchResult?
```

#### 일치 여부 판단

- 패턴과 문자열이 완전히 일치하는지 여부 확인

```kotlin
fun matches(input: CharSequence): Boolean
```

- 주어진 index에서 패턴과 문자열이 완전히 일치하는 부분 반환

```kotlin
fun matchesAt(input: CharSequence, index: Int): Boolean
```

- 패턴과 일치하는 부분이 있는지 확인

```kotlin
fun containsMatchIn(input: CharSequence): Boolean
```

#### 가공

- 패턴과 일치하는 부분 문자열 치환

```kotlin
fun replace(input: CharSequence, replacement: String): String
```

- 패턴과 일치하는 부분을 기준으로 문자열 분할

```kotlin
fun split(input: CharSequence, limit: Int = 0): List<String>
```

## 정규 표현식 패턴 옵션

- IGNORE_CASE : 대소문자 구분 X
- LITERAL : 메타데이터와 이스케이프 문자가 가진 의미를 무시
- COMMENTS – 공백과 주석을 무시

## 정규 표현식

#### Character classes

- [abc] : a, b, c
- [^abc] : a, b, c를 제외한 문자
- [a-zA-Z] : a ~ z, A ~ Z 문자 포함
- [a-dm-p] : a ~ d, m ~ p 문자 포함

#### Predefined character classes

- \d : [0-9]
- \D : [^0-9]
- \w : [a-zA-Z_0-9]
- \W : [^\w]

#### Greedy quantifiers

- X? : X가 없거나 하나
- X* : X가 없거나 하나 이상
- X+ : X가 하나 이상
- X{n} : X가 정확히 n번
- X{n,} : X가 적어도 n번
- X{n, m} : 적어도 n번 이상, m번 이하

#### Logical operators

- XY : X 다음 Y
- X\|Y : X 또는 Y
- (X) : X 그룹화. (abc)+ : **abcabc**bac

> 공식 문서 : [https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html)
> 
> 정규 표현식 테스트 : [https://www.regexplanet.com/advanced/java/index.html](https://www.regexplanet.com/advanced/java/index.html)
> 
> 정규 표현식 테스트 및 시각화 : [https://softwium.com/javex/](https://softwium.com/javex/)

## References

- [https://www.baeldung.com/kotlin/regular-expressions](https://www.baeldung.com/kotlin/regular-expressions)
