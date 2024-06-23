---
layout: post
title: MultipleBagFetchException
categories: jpa
tags: jpa
---

> org.hibernate.loader.MultipleBagFetchException: cannot simultaneously fetch multiple bags: [team.plot.core.entity.user.User.interests, team.plot.core.entity.user.User.personalities]

`Bag` 이라는 개념이 등장하는데, 이는 `List`와 유사하며 Hibernate 용어이다. `List`처럼 중복은 허용하지만, 순서는 보장하지 않는다.

Java Collection Framework에서는 중복을 허용하지만, 순서를 보장하지 않는 컬렉션은 존재하지 않는다. List는 중복을 허용하며 순서를 보장하지만, Hibernate는 이를 순서를 보장하지 않는 방식으로 처리한다.

> [https://zrr.kr/qIzs](https://zrr.kr/qIzs)

```java
// Bag
private List<T> collection;

// List
@OrderColumn(name = "position")
private List<T> collection;
```

**발생 원인**

단일 JPQL 쿼리에서 두 개의 컬렉션을 동시에 가져오려고 하면 Exception이 발생하게 된다.

```java
@Entity
class Artist {
    @OneToMany(mappedBy = "artist", fetch = FetchType.EAGER)
    private List<Song> songs;

    @OneToMany(mappedBy = "artist", fetch = FetchType.EAGER)
    private List<Offer> offers;
}
```

해당 쿼리는 카디션 곱을 발생시킨다. 이때 쿼리 결과는 하나의 큰 결과 집합으로 반환되며, 순서가 없는 `Bag`은 올바른 엔티티랑 매핑할 수 없다.

```sql
SELECT a.*, s.*, o.*
FROM Artist a
LEFT JOIN Song s ON a.id = s.artist_id
LEFT JOIN Offer o ON a.id = o.artist_id
```

**해결 방법**

1.	컬렉션 타입을 `List`가 아닌 `Set`으로 변경한다. 컬렉션에 순서나 중복된 요소가 필요하지 않은 경우 적절한 선택이다.
2.	두 개 이상의 JPQL 쿼리로 나누어 조회한다.

> [https://www.baeldung.com/java-hibernate-multiplebagfetchexception](https://www.baeldung.com/java-hibernate-multiplebagfetchexception)