---
layout: post
title: Cosine Similarity
categories: math
tags: math
---

코사인 유사도(cosine similarity)는 내적공간의 두 벡터 간 각도의 코사인값을 이용하여 측정된 벡터 간의 유사한 정도를 의미한다. 벡터 간 유사도를 측정하는 것이기 때문에 크기는 상관없고 방향만 활용한다. 적용할 수 있는 데이터셋은 이산 변수에 적합하다.

수식은 `두 벡터의 내적 / (A 벡터의 크기 * B 벡터의 크기)` 과 같다.

![cosine-similarity](/assets/postImages/CosineSimilarity/cosine-similarity.svg)

> **벡터 크기 공식 (Vector Norm)**
>
> ![vector-norm](/assets/postImages/CosineSimilarity/vector-norm.svg)

코사인 유사도 값은 -1 ~ 1 사이의 값을 가지고 1에 가까울수록 유사도가 높다.

- 유사도 값 > 0 : 같은 방향
- 유사도 값 = 0 : 수직 방향
- 유사도 값 < 0 : 반대 방향

이런 코사인 유사도는 추천 시스템에서 사용될 수 있다.

#### 구현 예제

- 사용자들의 상품 구매 데이터를 기반으로 서로의 구매 패턴의 유사도를 파악할 수 있다.

```kotlin
fun cosineSimilarity(vec1: List<Double>, vec2: List<Double>): Double {
    require(vec1.size == vec2.size) { "Vectors must be of the same length" }

    // 두 벡터의 내적
    val dotProduct = vec1.zip(vec2).sumOf { it.first * it.second }

    // 두 벡터의 크기
    val normVec1 = sqrt(vec1.sumOf { it * it })
    val normVec2 = sqrt(vec2.sumOf { it * it })

    // 코사인 유사도 계산
    return dotProduct / (normVec1 * normVec2)
}

fun main() {
    val user1 = listOf(1.0, 1.0, 0.0, 0.0, 1.0)
    val user2 = listOf(1.0, 1.0, 0.0, 0.0, 1.0)
    val user3 = listOf(0.0, 1.0, 0.0, 1.0, 1.0)
    val user4 = listOf(1.0, 1.0, 0.0, 0.0, 0.0)

    val similarityWithUser2 = cosineSimilarity(user1, user2)
    val similarityWithUser3 = cosineSimilarity(user1, user3)
    val similarityWithUser4 = cosineSimilarity(user1, user4)

    println("사용자 1과 사용자 2의 유사도: %.4f".format(similarityWithUser2)) // 1.0000
    println("사용자 1과 사용자 3의 유사도: %.4f".format(similarityWithUser3)) // 0.6667
    println("사용자 1과 사용자 4의 유사도: %.4f".format(similarityWithUser4)) // 0.8165
}
```

> [https://en.wikipedia.org/wiki/Cosine_similarity](https://en.wikipedia.org/wiki/Cosine_similarity)
> 
> [https://ko.wikipedia.org/wiki/유클리드_벡터](https://ko.wikipedia.org/wiki/유클리드_벡터)