---
title: "[JPA] @DynamicInsert, @DynamicUpdate 실습"
description: "@DynamicInsert, @DynamicUpdate를 사용해보고 성능을 비교해보자"
categories:
 - JPA
tags:
 - SpringBoot
 - Spring Data JPA
 - Entity
---

# 쓰게된 이유

<hr>

``` java
create table users {
  reg_date DATETIME default CURRENT_TIMESTAMP,
}

public class UserEntity {
  ...
  @ColumnDefault(value = "CURRENT_TIMESTAMP")
      private LocalDateTime regDate;
}

userRepository.save(userEntity)
```

DB 테이블과 엔티티에 유저 등록일(regDate)의 default 값을 현재 시간으로 설정하고 더미 데이터를 추가하니 아래와 같이 시간이 아닌 null 값이 들어가있는 것을 확인했다.

![regDateNull](/assets/postImages/EntityDynamicQuery/regDateNull.JPG)

![mid_spaceout](/assets/postImages/common/mid_spaceout.jpg)

원인이 무엇일까 찾아보니 JPA는 기본적으로 변경 감지하여 update 쿼리를 날리게 되면 수정된 컬럼만 수정하는 쿼리문을 날리는 것이 아닌 전체 컬럼을 수정하는 쿼리문을 날리게 된다. 이때 regDate은 임의로 넣어두지 않았기 때문에 null으로 들어가있어서 null로 저장이 되는 것이었다.

# 해결 방안

<hr>

## 강제로 값 삽입

``` java
UserEntity userEntity = UserEntity.builder()
                          ...
                          .regDate(LocalDateTime.now())
                          .build();

userRepository.save(userEntity)
```

삽입 또는 수정하기 전에 regDate에 값을 넣는 방식이다.

## @PrePersist, @PreUpdate를 사용하는 방법

``` java
@PrePersist
public void initPersist() {
  this.regDate = LocalDateTime.now();
}
```

삽입 또는 수정하기 전에 수행할 로직을 작성하여 값을 정의하는 방식이다.

## @DynamicInsert, @DynamicUpdate를 사용하는 방법

해당 어노테이션을 엔티티에 적용시켜 삽입 또는 수정 쿼리를 동적으로 만드는 방식이다. 이 방식이 이번에 사용해 볼 내용이다.

# 사전 작업

<hr>

## Entity

``` java
@Entity @Getter
@Table(name = "users")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String columnA;
    private String columnB;
    private String columnC;
    private String columnD;
    private String columnE;
    private String columnF;
    private String columnG;
    private String columnH;
    private String columnI;
    private String columnJ;
    private String columnK;
    private String columnL;
    private String columnM;

    public User(String columnA) {
        this.columnA = columnA;
    }

    public void changeColumnA(String columnA) {
        this.columnA = columnA;
    }
}
```

## Repository

``` java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

# @DynamicInsert

<hr>

이 어노테이션을 적용하게 되면 `Insert` 쿼리를 날릴 때 null인 값은 제외하고 쿼리문이 만들어진다.

## 적용 방법

``` java
@DynamicInsert
public class User {
  ...
}
```

## 적용 전

![dynamicInsertQuery](/assets/postImages/EntityDynamicQuery/dynamicInsertQuery.JPG)

## 적용 후

![dynamicInsertQuery2](/assets/postImages/EntityDynamicQuery/dynamicInsertQuery2.JPG)

## 적용 전/후 비교

``` java
@RepeatedTest(5)
public void 유저생성() throws Exception {
    //given
    long start = System.currentTimeMillis();
    //when
    for (int i = 0; i < 50000; i++) {
        userRepository.save(new User("columnA"));
    }
    //then
    System.out.println((System.currentTimeMillis() - start) / 1000 + "s");
}
```

![inserCompareTime](/assets/postImages/EntityDynamicQuery/inserCompareTime.JPG)

테스트 코드를 작성하여 `@DynamicInsert`를 적용한 것과 안한 것을 5번 씩 실행한 결과이다. 사진에서 보면 알 수 있듯이 적용한 코드가 빠른 것을 볼 수 있다.

# @DynamicUpdate

<hr>

이 어노테이션을 적용하게 되면 `Update` 쿼리를 날릴 때 null인 값은 제외하고 쿼리문이 만들어진다.

## 적용 방법

``` java
@DynamicUpdate
public class User {
  ...
}
```

## 적용 전

![dynamicUpdateQuery](/assets/postImages/EntityDynamicQuery/dynamicUpdateQuery.JPG)

## 적용 후

![dynamicUpdateQuery2](/assets/postImages/EntityDynamicQuery/dynamicUpdateQuery2.JPG)

## 적용 전/후 비교

``` java
@RepeatedTest(5)
public void 유저수정() throws Exception {
    //given
    User user = userRepository.save(new User("columnA"));
    long start = System.currentTimeMillis();
    //when
    for (int i = 0; i < 50000; i++) {
        user.changeColumnA("columnA" + i);
        entityManager.flush();
    }
    //then
    System.out.println((System.currentTimeMillis() - start) / 1000 + "s");
}
```

![updateCompareTime](/assets/postImages/EntityDynamicQuery/updateCompareTime.JPG)

테스트 코드를 작성하여 `@DynamicUpdate`를 적용한 것과 안한 것을 5번 씩 실행한 결과이다. 사진에서 보면 알 수 있듯이 적용한 코드가 빠른 것을 볼 수 있다.

# 결론

<hr>

`@DynamicInsert`, `@DynamicUpdate` 를 사용하게 되면 불필요한 DB 부하를 줄일 수 있고, default 값 대신에 null 값이 들어갈 일은 없을 것이다.

테이블에 컬럼 개수가 많다면, Default 값에 null 값이 들어갈 우려가 있다면 해당 어노테이션을 쓰는 것을 고려해보자!
