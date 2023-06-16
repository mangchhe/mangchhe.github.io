---
title: "Is it possible to chage the value of primary and foreign keys?"
categories: db
tags: db
---

# 사전 준비

<hr>

## 실습 환경

```bash
mysql 8.0.27
```

## 테이블 생성

![exERD](/assets/postImages/PrimaryForeignKeyUpdate/exERD.png)

```sql
create table users(
    user_id int not null auto_increment,
    name varchar(100),
    primary key(user_id)
);

create table board(
    board_id int not null auto_increment,
    title varchar(100),
    content varchar(100),
    user_id int,
    primary key(board_id)
);
```

## 데이터 삽입

![exTableData](/assets/postImages/PrimaryForeignKeyUpdate/exTableData.png)

```sql
insert into users(name) values('hajoo1');
insert into users(name) values('hajoo2');

insert into board(title, content, user_id) values('title1', 'content1', 1);
insert into board(title, content, user_id) values('title2', 'content2', 2);
```

# non-relational

<hr>

- 외래키 설정 X

![usersUpdate3](/assets/postImages/PrimaryForeignKeyUpdate/usersUpdate3.png){: width="300" }

- users 테이블 user_id를 2 -> 3으로 변경
- 기본키는 연관 관계가 없다면 **변경이 가능**하다.

```
update users set user_id = 3 where name = 'hajoo2';
```

# non-cascade

<hr>

- 외래키 설정 O, cascade X

## 외래키 설정

```sql
alter table board add constraint ex_fk foreign key(user_id) references users(user_id);
```

## users 기본키 변경

![userUpdate](/assets/postImages/PrimaryForeignKeyUpdate/usersUpdate.png){: width="300" }

- users 테이블 user_id를 2 -> 3으로 변경
- 연관 관계에 있는 부모 행은 **변경이나 삭제가 불가능**하다고 나온다.

```sql
update users set user_id = 3 where name = 'hajoo2';
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`test`.`board`, CONSTRAINT `ex_fk` FOREIGN KEY (`user_id`) REFERENCES `users` (`user_id`))
```

## board 외래키 변경

![boardUpdate](/assets/postImages/PrimaryForeignKeyUpdate/boardUpdate.png)

- board 테이블 user_id를 2 -> 3으로 변경
- 연관 관계에 있는 부모 행은 **변경이나 삭제가 불가능**하다고 나온다.

```sql
update board set user_id = 3 where title = 'title2';
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`test`.`board`, CONSTRAINT `ex_fk` FOREIGN KEY (`user_id`) REFERENCES `users` (`user_id`))
```

![boardUpdate4](/assets/postImages/PrimaryForeignKeyUpdate/boardUpdate4.png)

- board 테이블 user_id를 2 -> 1로 변경
- board 테이블 user_id만 1로 **변경됨**

```sql
update board set user_id = 1 where title = 'title2';
```

- 외래키의 경우에 부모 테이블에 존재하는 키일 경우에는 변경이 가능하다.

# update-cascade

<hr>

- 외래키 설정 O, cascade O

## 외래키 설정

```sql
alter table board add constraint ex_fk foreign key(user_id) references users(user_id) on update cascade;
```

## users 기본키 변경

![usersUpdate2](/assets/postImages/PrimaryForeignKeyUpdate/usersUpdate2.png)

- users 테이블 user_id를 2 -> 3으로 변경
- users, board 테이블 user_id가 함께 변경됨

```sql
update users set user_id = 3 where name = 'hajoo2';
```

## board 외래키 변경

![boardUpdate2](/assets/postImages/PrimaryForeignKeyUpdate/boardUpdate2.png)

- board 테이블 user_id를 3 -> 4로 변경
- 연관 관계에 있는 부모 행은 **변경이나 삭제가 불가능**하다고 나온다.

```sql
update board set user_id = 4 where title = 'title2';
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`test`.`board`, CONSTRAINT `ex_fk` FOREIGN KEY (`user_id`) REFERENCES `users` (`user_id`) ON UPDATE CASCADE)
```

![boardUpdate3](/assets/postImages/PrimaryForeignKeyUpdate/boardUpdate3.png)

- board 테이블 user_id를 3 -> 1로 변경
- board 테이블 user_id만 1로 **변경이 됨**

```sql
update board set user_id = 1 where title = 'title2';
```

- 외래키의 경우에 부모 테이블에 존재하는 키일 경우에는 변경이 가능하다.