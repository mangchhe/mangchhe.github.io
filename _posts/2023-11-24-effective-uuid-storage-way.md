---
layout: post
title: Effective UUID storage way from Mysql
categories: cs
tags: cs
---

여러 데이터베이스에서 특정 콘텐츠의 식별을 통합하기 위해 PK로 UUID로 사용하고 있다.

일반적으로 사용하는 auto increment pk가 아닌 UUID를 사용하게 되면 몇 가지 문제점이 존재한다.

`BIGINT`는 [8byte](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)인데 UUID에서 사용하는 타입 `CHAR`는 [36byte](https://dev.mysql.com/doc/refman/8.0/en/char.html)이다. 그에 따라 인덱스 생성 등에 더 큰 공간을 요구하게 된다.

또한 생성되는 UUID가 sequential한 값이 아니라서 insert 시에 계속 자리 교체가 발생하여 성능 저하가 발생할 수 있다.

## Solution

**값의 커진 공간을 줄이기 위해** 구분자(-)를 모두 제거하고 바이너리 형태로 변경하여 저장하면 공간을 줄이는 것이 가능해진다. 36byte에서 구분자 총 4byte를 제거하고 32byte의 각 문자는 16진수, 즉 하나당 4bit를 나타내니까 총 128bit로 16byte BINARY(16)으로 저장할 수 있다.

**sequential 한 값을 만들기 위해** uuid에 포함되어 있는 timestamp 값을 활용한다. 만약 없다고 한다면 추가해서 하는 방법을 고려해야 한다. 

v1의 경우 **58e0a7d7–eebc–11d8**-9669-0800200c9a66 앞에 세 개의 섹션의 uuid가 timestamp를 가르킨다. 좀 더 sequential한 정렬을 만들기 위해서 ⓷-⓶-⓵ 섹션으로 UUID를 11d8-eebc-58e0a7d7로 재정렬을 한다.

## Implementation

의존성 : [github](https://github.com/cowtowncoder/java-uuid-generator)

```gradle
implementation("com.fasterxml.uuid:java-uuid-generator:4.3.0")
```

다음과 같이 

```kotlin
fun generateUUID(): String {
    val uuid = Generators.timeBasedGenerator().generate()
    val uuidSections = uuid.toString().split("-")
    return uuidSections[2] + uuidSections[1] + uuidSections[0] + uuidSections[3] + uuidSections[4]
}
```

## References

- [https://en.wikipedia.org/wiki/Universally_unique_identifier](https://en.wikipedia.org/wiki/Universally_unique_identifier)
- [https://www.percona.com/blog/store-uuid-optimized-way/#crayon-60fa2fbab27f7557869434](https://www.percona.com/blog/store-uuid-optimized-way/#crayon-60fa2fbab27f7557869434)