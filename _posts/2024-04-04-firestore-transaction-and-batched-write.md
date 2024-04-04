---
layout: post
title: Firestore Transaction and Batched Write
categories: db
tags: db
---

Firestore를 사용하다 보면, 때때로 단일 작업 처리 중에 문제가 발생해 일부 작업만 적용되는 상황에 직면할 수 있다. 이런 경우 일부 작업의 롤백이 필요한데 이는 까다로운 작업일 수 있다.

이러한 문제에 대한 해결책으로 Firestore는 여러 작업을 단일 트랜잭션으로 묶는 기능을 제공한다.

트랜잭션 내에서 `get()`, `set()`, `update()`, `delete()`를 포함하고 트랜잭션은 쓰기 작업을 부분적으로 적용하지 않음으로써 **원자성과 정합성을 보장**한다.

#### 트랜잭션 사용 시 참고 사항

- **읽기-쓰기 순서**: 트랜잭션에서 읽기 작업은 반드시 쓰기 작업 전에 이루어져야 한다.
- **동시성 관리**: 트랜잭션이 읽는 문서가 다른 곳에서 동시에 수정되는 경우 트랜잭션을 실행하는 함수가 여러 번 재실행될 수 한다.
- **애플리케이션 상태와의 독립성**: 트랜잭션 함수는 애플리케이션 상태를 직접 수정해서는 안 된다. 이는 데이터베이스의 일관성과 정합성을 유지하는 데 필수적이다.
- **오프라인 상태의 한계**: 클라이언트가 오프라인일 경우 트랜잭션은 실패한다. 이는 Firestore 트랜잭션이 실시간 통신에 의존하기 때문이다.

#### 트랜잭션 실패 원인

- 읽기 작업이 쓰기 작업 후에 수행되는 경우
- 트랜잭션이 트랜잭션 외부에서 수정된 문서를 읽는 경우
- 트랜잭션의 요청 크기가 최대 10MiB를 초과하는 경우
- 트랜잭션 내의 작업 수가 500개를 초과하는 경우

#### 사용법

```java
WriteBatch batch = db.batch();

DocumentReference nycRef = db.collection("cities").document("NYC");
batch.set(nycRef, new City());

DocumentReference sfRef = db.collection("cities").document("SF");
batch.update(sfRef, "population", 1000000L);

DocumentReference laRef = db.collection("cities").document("LA");
batch.delete(laRef);

batch.commit().addOnCompleteListener(new OnCompleteListener<Void>() {
    @Override
    public void onComplete(@NonNull Task<Void> task) {
        // ...
    }
});
```

> [https://firebase.google.com/docs/firestore/manage-data/transactions?hl=en](https://firebase.google.com/docs/firestore/manage-data/transactions?hl=en)