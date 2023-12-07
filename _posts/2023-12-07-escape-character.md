---
layout: post
title: Escape character
categories: cs
tags: cs
---

이스케이프 문자는 문자 시퀀스 다음 문자에 대해 다른 해석을 나타내는 특별한 문자이다.

이스케이프 자체에 의미를 가지지 않고 두 개 이상의 문자 시퀀스로 이루어져 있고 여럿 프로그래밍 언어와 데이터 포맷, 통신 프로토콜의 문맥 일부로 사용된다.

#### JavaScript

- `\b` : 백스페이스
- `\t` : 탭
- `\r` : 캐리지 리턴 (커서 맨 앞으로 이동)
- `\n` : 줄바꿈

```kotlin
println("\'") // '
println("\"") // "
println("\\") // \
println("\n") // 줄바꿈 
println("a\rbc") // bc
println("a\tb") // a	b
println("ab\bc") // ac
```

#### Bourne shell

```sh
rm *    # delete all files in the current directory
rm \*   # delete the file named *
```

## References

- [https://en.wikipedia.org/wiki/Escape_character](https://en.wikipedia.org/wiki/Escape_character)