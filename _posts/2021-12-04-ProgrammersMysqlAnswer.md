---
layout: post
title: "Programmer SQL high score kit correct answer"
categories: db
tags: db
---

# SELECT

## 모든 레코드 조회하기

``` sql
SELECT *
from animal_ins
order by animal_id
```

## 역순 정렬하기

``` sql
SELECT name, datetime
from animal_ins
order by animal_id desc
```

## 아픈 동물 찾기

``` sql
SELECT animal_id, name
from animal_ins
where intake_condition = 'sick'
order by animal_id
```

## 어린 동물 찾기

``` sql
SELECT animal_id, name
from animal_ins
where intake_condition != 'Aged'
order by animal_id
```

## 동물의 아이디와 이름

``` sql
SELECT animal_id, name
from animal_ins
order by animal_id
```

## 여러 기준으로 정렬하기

``` sql
SELECT animal_id, name, datetime
from animal_ins
order by name, datetime desc
```

## 상위 n개 레코드

``` sql
SELECT name
from animal_ins
order by datetime
limit 1
```

# SUM, MAX, MIN

## 최댓값 구하기

``` sql
select max(datetime) as "시간"
from animal_ins
```

## 최솟값 구하기

``` sql
SELECT min(datetime) as '시간'
from animal_ins
```

## 동물 수 구하기

``` sql
SELECT count(*)
from animal_ins
```

## 중복 제거하기

``` sql
SELECT count(distinct name)
from animal_ins
```

# GROUP BY

## 고양이와 개는 몇 마리 있을까

``` sql
SELECT animal_type, count(animal_type)
from animal_ins
group by animal_type
order by animal_type
```

## 동명 동물 수 찾기

``` sql
SELECT name, count(name)
from animal_ins
group by name
having count(name) > 1
order by name
```

## 입양 시각 구하기(1)

``` sql
SELECT hour(datetime) 'HOUR', count(datetime) 'count'
from animal_outs
group by HOUR
having HOUR between 9 and 19
order by HOUR
```

## 입양 시각 구하기(2)

``` sql
with recursive tmp_table as (
    select 0 as h
    union all
    select h+1 from tmp_table where h < 23
)

SELECT h, count(animal_id)
from tmp_table
left outer join animal_outs
on hour(datetime) = tmp_table.h
group by h
order by h
```

# IS Null

## 이름이 없는 동물의 아이디

``` sql
SELECT animal_id
from animal_ins
where name = null
```

## 이름이 있는 동물의 아이디

``` sql
SELECT animal_id
from animal_ins
where name is not null
```

## NULL 처리하기

``` sql
SELECT animal_type, ifnull(name, 'No name'), sex_upon_intake
from animal_ins
order by animal_id
```

# JOIN

## 없어진 기록 찾기

``` sql
SELECT o.animal_id, o.name
from animal_outs o
left outer join animal_ins i
on o.animal_id = i.animal_id
where o.animal_id is not null and i.animal_id is null
```

## 있었는데요 없었습니다

``` sql
SELECT i.animal_id, i.name
from animal_ins i
inner join animal_outs o
on i.animal_id = o.animal_id
where o.datetime < i.datetime
order by i.datetime
```

## 오랜 기간 보호한 동물(1)

``` sql
SELECT i.name, i.datetime
from animal_ins i
left outer join animal_outs o
on i.animal_id = o.animal_id
where o.animal_id is null
order by i.datetime
limit 3
```

## 보호소에서 중성화한 동물

``` sql
SELECT i.animal_id, i.animal_type, i.name
from animal_ins i
inner join animal_outs o
on i.animal_id = o.animal_id
where i.sex_upon_intake like 'Intact%' 
and (o.sex_upon_outcome like 'Spayed%' oro.sex_upon_outcome like 'Neutered%')
order by i.animal_id
```

# String, Data

## 루시와 엘라 찾기

``` sql
SELECT animal_id, name, sex_upon_intake
from animal_ins
where name in ('Lucy', 'Ella', 'Pickle', 'Rogan', 'Sabrina', 'Mitty')
```

## 이름이 el이 들어가는 동물 찾기

``` sql
SELECT animal_id, name
from animal_ins
where animal_type = 'dog' and name like '%el%'
order by name
```

## 중성화 여부 파악하기

``` sql
SELECT animal_id ANIMAL_ID, name NAME,
case when sex_upon_intake like '%Neutered%' or sex_upon_intake like '%Spayed%'
then 'O' else 'X' end 중성화
from animal_ins
```

## 오랜 기간 보호한 동물(2)

``` sql
SELECT i.animal_id, i.name
from animal_ins i
inner join animal_outs o
on i.animal_id = o.animal_id
order by o.datetime - i.datetime desc
limit 2
```

## DATETIME에서 DATE로 형 변환

``` sql
SELECT animal_id, name, date_format(datetime, '%Y-%m-%d')
from animal_ins
order by animal_id
```

### Reference

[https://programmers.co.kr/learn/challenges](https://programmers.co.kr/learn/challenges)