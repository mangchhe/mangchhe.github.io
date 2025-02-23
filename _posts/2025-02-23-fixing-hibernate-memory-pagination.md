---
layout: post
title: Fixing Hibernate Memory Pagination
categories: jpa
tags: spring, jpa, orm, datadog
---

단순 조회 API에서 예상치 못한 높은 지연 시간이 발생했다. 초기에는 쿼리 성능 문제로 의심했으나, 실제 로그 분석 결과 Hibernate의 페이징 처리 방식에서 문제가 발견되었다.

> HHH90003004: firstResult/maxResults specified with collection fetch; applying in memory

이 경고 로그는 Hibernate가 페이징 처리를 메모리에서 수행하고 있다는 것을 나타낸다.

Collection fetch join을 사용하면서 동시에 페이징을 적용할 때 Hibernate가 DB에서 모든 데이터를 가져온 후 메모리에서 페이징을 수행한다고 한다.

#### 원본 코드

```kotlin
JPAQuery<User> query = queryFactory.selectFrom(user)
        .leftJoin(user.files, userFile).fetchJoin()
        .leftJoin(user.userInflow, userInflow).fetchJoin()
        .leftJoin(user.personalities, userPersonality).fetchJoin()
        .leftJoin(user.interests, userInterest).fetchJoin()
        .leftJoin(user.managerUserEvaluation, managerUserEvaluation).fetchJoin()
        .where(booleanBuilder)
        .orderBy(user.activatedAt.desc());

int total = query.fetch().size();
List<User> results = query.offset(pageable.getOffset())
        .limit(pageable.getPageSize())
        .fetch();

return new PageImpl<>(results, pageable, total);
```

문제의 원본 코드이다.
- OneToMany 관계(files, personalities, interests)에 대한 fetch join 사용
- 동일한 쿼리로 전체 카운트와 결과를 조회
- 페이징 처리가 메모리에서 발생

#### 개선된 코드

```kotlin
JPAQuery<User> query = queryFactory
        .selectFrom(user)
        .leftJoin(user.userInflow, userInflow).fetchJoin()
        .leftJoin(user.managerUserEvaluation, managerUserEvaluation).fetchJoin()
        .where(booleanBuilder)
        .orderBy(user.activatedAt.desc());

JPAQuery<Long> countQuery = queryFactory
        .select(user.count())
        .from(user)
        .where(booleanBuilder);

List<User> results = query
        .offset(pageable.getOffset())
        .limit(pageable.getPageSize())
        .fetch();

Long totalCount = countQuery.fetchOne();
return new PageImpl<>(results, pageable, totalCount != null ? totalCount : 0L);
```

아래와 같은 방식으로 해당 문제를 해결할 수 있었다.

- OneToMany 관계에 대한 fetch join 제거
- 카운트 쿼리 분리
- 데이터베이스 수준에서 페이징이 동작하도록 수정

#### 성능 개선 전후 비교

- 응답 시간이 초 단위에서 밀리초 단위로 크게 개선되었고, 에러도 완전히 해소되었다.

![apm](/assets/postImages/FixingHibernateMemoryPagination/apm.png)

> [https://jojoldu.tistory.com/737](https://jojoldu.tistory.com/737)