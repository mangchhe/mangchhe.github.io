---
layout: post
title: EXPLAIN Statement
categories: db
tags: db
---

`EXPLAIN` 구문은 어떻게 명령문을 실행하는지에 대한 정보를 제공한다.

```sh
{EXPLAIN | DESCRIBE | DESC}
    tbl_name [col_name | wild]

{EXPLAIN | DESCRIBE | DESC}
    [explain_type]
    {explainable_stmt | FOR CONNECTION connection_id}

{EXPLAIN | DESCRIBE | DESC} ANALYZE [FORMAT = TREE] select_statement

explain_type: {
    FORMAT = format_name
}

# 실행 계획 포맷 형식
format_name: {
    TRADITIONAL
  | JSON
  | TREE
}

# 실행 가능한 구문
explainable_stmt: {
    SELECT statement
  | TABLE statement
  | DELETE statement
  | INSERT statement
  | REPLACE statement
  | UPDATE statement
}
```

> [https://dev.mysql.com/doc/refman/8.0/en/explain.html](https://dev.mysql.com/doc/refman/8.0/en/explain.html)

## EXPLAIN Output Format

Optimizer 가 명령문 실행 계획에 대해서 출력하고 실행 계획은 어떻게 명령문이 진행되는지 테이블 조인 방법과 순서에 대해서 알려준다.

| Column        | Json name     | Meaning           |
|---------------|---------------|-------------------|
| id            | select_id     | SELECT 쿼리 식별자     |
| select_type   | None          | SELECT 쿼리 타입      |
| table         | table_name    | 테이블명              |
| partitions    | partitions    | 일치하는 파티션          |
| type          | access_type   | 조인 타입             |
| possible_keys | possible_keys | 선택 가능한 인덱스        |
| key           | key           | 실제로 선택된 인덱스       |
| key_len       | key_length    | 선택된 인덱스의 길이       |
| ref           | ref           | 인덱스와 비교된 컬럼       |
| rows          | rows          | 검사된 행 추정치         |
| filtered      | filtered      | 테이블 조건에 필터링된 행 개수 |
| Extra         | None          | 추가적인 정보           |

#### id

`SELECT` 식별자이고 쿼리 내에 시퀀스한 번호이다. 만약 행이 다른 행의 결과와 union 참조면 이 값은 NULL 일 수 있다. 이 경우 테이블 컬럼은 `<unionM,N>`와 같이 표시된다.

#### select_type

| Value                | Meaning                                          |
|----------------------|--------------------------------------------------|
| SIMPLE               | 단순 조회 쿼리                                         |
| PRIMARY              | 가장 바깥 쿼리                                         |
| UNION                | UNION 구문에서 두 번째 이후 쿼리                            |
| DEPENDENT UNION      | outer query 에 의존하는 UNION 구문에서 두 번째 이후 쿼리         |
| UNION RESULT         | UNION 결과                                         |
| SUBQUERY             | 첫 번째 서브 쿼리                                       |
| DEPENDENT SUBQUERY   | outer query 에 의존하는 첫 번째 서브 쿼리                    |
| DERIVED              | 임시로 생성된 테이블                                      |
| DEPENDENT DERIVED    | 다른 테이블에 의존하는 임시로 생성된 테이블                         |
| MATERIALIZED         | 서브 쿼리의 결과를 임시로 저장하는 테이블                          |
| UNCACHEABLE SUBQUERY | 서브 쿼리의 결과를 캐싱하지 않고 재계산                           |
| UNCACHEABLE UNION    | UNION 구문에서 두 번째 이후 쿼리에 unreachable subquery 가 포함 |

#### table

출력 행이 참조하는 테이블의 이름이다. `<unionM,N>`, `<derivedN>`, `<subqueryN>` 종류를 가진다.

#### partitions

쿼리와 일치하는 레코드가 있는 파티션이다. 파티션이 없는 테이블의 경우에는 NULL 값이다.

#### type

| Value  | Meaning                                                                 |
|--------|-------------------------------------------------------------------------|
| system | 테이블이 하나의 행만 가질 경우 조인 타입                                                 |
| const  | 테이블에서 쿼리를 처음부터 읽었을 때 최대 일치하는 행이 하나일 경우 조인 타입. pk, unique key로 비교할 때 사용된다. |
| eq_ref | 테이블에서 한 행을 읽는다.                                                         |
|        |                                                                         |
|        |                                                                         |
|        |                                                                         |
|        |                                                                         |




> [https://dev.mysql.com/doc/refman/8.0/en/explain-output.html](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html)

## References

