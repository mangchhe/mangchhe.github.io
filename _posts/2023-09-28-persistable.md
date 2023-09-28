---
layout: post
title: Persistable
categories: jpa
tags: jpa
---

JPA를 사용할 때 `@GeneratedValue` 자동 생성 전략을 사용하지 않고 직접 식별키(PK)를 할당하여 INSERT를 하려고 하면 문제가 생긴다.

```kotlin
@Entity
class Member(
    @Id
    var id: Long = 0L,

    var name: String,
)

memberRepository.save(Member(id = 1L, name = "name2"))
```

다음과 같이 Member를 만들고 생성하려고 하면 select 쿼리가 날아간 다음에 insert 쿼리가 발생하는 것을 확인할 수 있다.

```sql
select
    m1_0.id,
    m1_0.name 
from
    member m1_0 
where
    m1_0.id=?
---
insert 
into
    member
    (name,id) 
values
    (?,?)
```

왜 그런지 디버깅을 해보자.

save 메서드 구현 부분을 보면 `entityInformation.isNew` 에서 해당 엔티티가 새로운 객체인지 판단한다.

```java
// SimpleJpaRepository
@Transactional
@Override
public <S extends T> S save(S entity) {

    Assert.notNull(entity, "Entity must not be null");

    if (entityInformation.isNew(entity)) {
        entityManager.persist(entity);
        return entity;
    } else {
        return entityManager.merge(entity);
    }
}
```

여기서 현재 구현체를 보게 되면 새로운 객체로 판단하는 경우
- primitive 타입일 경우에 pk가 null일 때
- Number 타입일 경우 0일 때

위 두 경우 제외하고는 예외를 발생시킨다.

여기서 자동 생성 전략이 아닌 직접 식별키를 할당할 경우 새로운 객체로 판단하지 않기 때문에 merge를 하게 된다. 이렇게 될 경우 준영속 상태로 판단하고 select 쿼리를 발생시켰으나 값이 존재하지 않아 insert를 하게 되는 것이다.

```java
// AbstractEntityInformation
public boolean isNew(T entity) {

    ID id = getId(entity);
    Class<ID> idType = getIdType();

    if (!idType.isPrimitive()) {
        return id == null;
    }

    if (id instanceof Number) {
        return ((Number) id).longValue() == 0L;
    }

    throw new IllegalArgumentException(String.format("Unsupported primitive id type %s", idType));
}
```

해결 방법은 엔티티에서 `Persistable`를 상속받아 `isNew` 함수를 오버라이딩하여 새로운 객체인지에 대해 판단하는 로직을 알맞게 변경하게 되면 계속해서 새로운 객체로 판단하여 불필요한 쿼리가 발생하는 것을 방지할 수 있다.

```kotlin
@Entity
class Member(
    @Id
    var id: Long = 0L,

    var name: String,
): Persistable<Long>
{
    override fun getId() = this.id

    override fun isNew() = this.id != -1L
}
```

### Reference

- [Spring Docs Persistable](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Persistable.html)