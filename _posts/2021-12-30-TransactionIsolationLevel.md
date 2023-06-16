---
title: "What is transaction isolation level"
categories: db
tags: db
---

# 트랜잭션 격리 수준이란?

<hr>

동시에 DB에서 여러 개의 트랜잭션을 처리할 때 **어느 정도 서로에게 고립되어있는지**를 말한다. 즉, 어떤 트랜잭션이 다른 트랜잭션의 변경된 데이터를 볼 수 있도록 허용할지 말지를 결정하는 것이다.

트랜잭션의 ACID 성질 중 I(Isolation)을 완벽하게 보장하기 위해서는 동시에 들어오는 트랜잭션에 대해서 큐 방식과 같이 차례대로 처리해나가야 한다. 이러한 방식으로 구현하게 될 때 동시성 처리의 성능이 매우 나빠지게 되므로 격리 수준을 나누게 되었다.

# 격리 종류

<hr>

격리 수준에 따라 총 네 가지로 나눠진다.

- Read Uncommitted
- Read Committed
- Repeatable Read
- Serializable

위에서 아래로 갈수록 격리 수준이 높아지기 때문에(더 엄격해짐) 동시에 들어오는 트랜잭션에 안전하지만, 격리 수준에 따른 문제점이 생길 수 있다.

## 격리 수준에 따른 문제점

격리 수준의 문제점은 총 세 가지가 있다.

- Dirty Read : 어떤 트랜잭션에서 처리한 작업이 완료되지 않았음에도 다른 트랜잭션이 볼 수 있는 현상
- Non-Repeatable Read : 동일한 조회 쿼리를 실행했을 때 항상 같은 결과를 보장하지 않는 현상
- Phantom Read : 동일한 조회 쿼리를 실행했을 때 이전에는 존재하지 않았던 유령 레코드가 나타나는 현상

이 세 가지 문제점은 각 격리 수준에 따라 다르게 나타난다.

|-|Dirty Read|Non-Repeatable read|Phantom Read|
|:-:|:-:|:-:||:-:|
|Read Uncommitted|O|O|O|
|Read Committed|-|O|O|
|Repeatable Read|-|-|O|
|Serializable|-|-|-|

## Read Uncommitted

커밋하지 않은 트랜잭션의 데이터 변경을 다른 트랜잭션에서 읽을 수 있도록 허용하는 것을 말한다.

![readUncommitted](/assets/postImages/TransactionIsolationLevel/readUncommitted.png){: width="700" }

트랜잭션A와 트랜잭션B가 동시에 수행되고 있다. 트랜잭션A에서 홍길동의 이름을 홍길동2로 변경하였고 commit/rollback을 하지 않았지만 트랜잭션B에서는 변경된 홍길동2의 데이터가 조회되는 것을 확인할 수 있다. 만약 사진처럼 트랜잭션A가 rollback으로 종료하게 된다면 트랜잭션B는 잘못된 값을 가지고 있는 것이기 때문에 정합성에 문제를 발생시킬 수 있다.

## Read Committed

커밋한 트랜잭션의 데이터 변경만을 다른 트랜잭션에서 읽을 수 있도록 허용하는 것을 말한다. Oracle DB에서 Default로 사용되는 격리 수준이다.

![readUncommitted](/assets/postImages/TransactionIsolationLevel/readCommitted.png){: width="700" }

이전과 다르게 커밋되기 전에는 이전 값인 홍길동이 그대로 출력이 되고 커밋된 이후에는 홍길동2가 나타나 dirty read가 해결된 것을 확인할 수 있다. 문제는 트랜잭션B 내에서 트랜잭션A의 커밋 전/후로 다른 데이터가 조회된다는 것이다. 

## Repeatable Read

트랜잭션 범위 내에서 조회한 데이터는 항상 동일한 데이터를 읽을 수 있도록 허용하는 것을 말한다. Mysql InnoDB에서 Default로 사용되는 격리 수준이다.

![readUncommitted](/assets/postImages/TransactionIsolationLevel/repeatableRead.png){: width="700" }

여기서는 이전까지 발생했던 dirty read, non-reapeatable read 문제는 나타나지 않는다. 하지만 이전에 조회했던 테이블에 값이 추가된다면 조회된 데이터의 불일치를 다시 발생하게 된다.

## Serializable

한 트랜잭션에서 사용하고 있는 데이터는 다른 트랜잭션에서 접근할 수 없는 것을 말한다.

![readUncommitted](/assets/postImages/TransactionIsolationLevel/serializable.png){: width="700" }

# Reference

<hr>

- [자바 ORM 표준 JPA 프로그래밍](https://book.naver.com/bookdb/book_detail.nhn?bid=9252528)
- [https://dar0m.tistory.com/225](https://dar0m.tistory.com/225)