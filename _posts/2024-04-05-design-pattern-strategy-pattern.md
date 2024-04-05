---
layout: post
title: Design Pattern - Strategy Pattern
categories: cs
tags: cs
---

여러 방식의 정렬을 구현하는 클래스를 만들어 사용할 때 전략 패턴을 사용하지 않는다면 아래와 같이 작성할 수 있다.

이 방법은 간단하지만, 확장성의 부재가 있다. 새로운 정렬 알고리즘을 추가할 때마다 `Sorter` 클래스를 수정해야 하므로 OCP를 위반한다.

```kotlin
class Sorter {
    fun sortArray(arr: Array<Int>, algorithmType: String): Array<Int> {
        return when (algorithmType) {
            "bubble" -> bubbleSort(arr)
            "quick" -> quickSort(arr)
             // 추가 정렬 알고리즘이 필요할 경우 여기에 코드 추가
            else -> arr
        }
    }

    private fun bubbleSort(arr: Array<Int>): Array<Int> {
        // 버블 정렬 로직 구현
        // ...
        return arr
    }

    private fun quickSort(arr: Array<Int>): Array<Int> {
        // 퀵 정렬 로직 구현
        // ...
        return arr
    }

     // 추가 정렬 알고리즘이 필요할 경우 여기에 메서드 추가
}
```

전략 패턴 디자인은 정렬 방법을 정의하고 각각을 캡슐화하여 이들을 교환 가능하게 만든다. 해당 알고리즘을 사용하는 클라이언트로부터 독립적으로 변화할 수 있게 해준다.

![strategy-pattern-diagram](/assets/postImages/DesignPatternStrategyPattern/strategy-pattern-diagram.png)

> [https://en.wikipedia.org/wiki/Strategy_pattern](https://en.wikipedia.org/wiki/Strategy_pattern)

우선 인터페이스를 만들고 모든 정렬 알고리즘이 구현할 `sort` 메서드를 선언한다.

```kotlin
interface SortingStrategy {
    fun sort(arr: Array<Int>): Array<Int>
}
```

버블 정렬과 퀵 정렬 알고리즘을 위한 구체적인 전략을 구현한다. 각 전략은 자체적으로 정렬 로직을 캡슐화하여 관심사를 분리한다.

```kotlin
class BubbleSortStrategy : SortingStrategy {
    override fun sort(arr: Array<Int>): Array<Int> {
        // 버블 정렬 알고리즘 구현
        // ...
        return arr
    }
}

class QuickSortStrategy : SortingStrategy {
    override fun sort(arr: Array<Int>): Array<Int> {
        // 퀵 정렬 알고리즘 구현
        // ...
        return arr
    }
}
```

마지막으로 클라이언트에서는 기존 코드 구조를 변경하지 않고도 동적으로 정렬 전략을 선택할 수 있게 된다.

```kotlin
fun main() {
    val array = arrayOf(3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5)

    var sorter: SortingStrategy = BubbleSortStrategy()
    sorter.sort(array)
    sorter = QuickSortStrategy()
    sorter.sort(array)
}
```

전락 패턴을 도입하면서 다양한 알고리즘을 쉽게 전환하거나 새로운 알고리즘을 도입할 때 기존 코드를 수정할 필요가 없어졌고 알고리즘을 그 자체 클래스로 캡슐화함으로써 코드 베이스가 관리하기 쉬워지고 오류 가능성을 줄일 수 있다.