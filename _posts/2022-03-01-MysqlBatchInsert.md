---
title: "[JPA] Batch Insert로 Insert 성능 향상시키기"
description: Mysql에서 identity, sequence 키 생성 전략에 따라 Batch Insert으로 인한 insert 성능 개선 방법
categories:
 - JPA
tags:
 - JPA
 - BatchInsert
---

# Batch Insert 란

<hr>

- 여러 개의 insert문을 하나로 통합하는 것을 말함
- 구문이 여러 개가 아닌 통합하여 하나만 보내기 때문에 성능 개선할 수 있음

```sql
// single insert
insert into table(column, column2) values(value1, value2);
insert into table(column, column2) values(value3, value4);
insert into table(column, column2) values(value5, value6);
insert into table(column, column2) values(value7, value8);

// batch insert
insert into table(column, column2) values(value1, value2), 
                                         (value3, value4),
                                         (value5, value6),
                                         (value7, value8);
```

# Sequence 전략에서의 Batch Insert

<hr>

## 적용방법

- `rewriteBatchedStatements` 옵션을 true 주고 `batch_size`를 지정해주게 되면 적용됨
- batch 사이즈는 한 번에 몇 개의 sql문을 묶을 지 결정

``` yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?rewriteBatchedStatements=true
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 1000
        #   order_inserts: true
        #   order_updates: true
```

### order_inserts, order_updates

- 같은 구문끼리 묶는데 사용이 됨
- 부모 엔티티 a1, a2를 넣고 자식 엔티티 b1, b2를 넣는 순서로 영속화가 진행이 된다면 아래와 같이 진행됨

```sql
// 옵션 적용 전
insert into A values(a1)
insert into B values(b1)
insert into A values(a2)
insert into B values(b2)

// 옵션 적용 후
insert into A values(a1, a2)
insert into B values(b1, b2)
```

## 준비 내용

```java
// Entity
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@SequenceGenerator(
    name = "BOARD_SEQ_GEN",
    sequenceName = "BOARD_SEQ",
    initialValue = 1,
    allocationSize = 60000
)
@Getter
public class Board {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "BOARD_SEQ_GEN")
    private Long id;

    private String title;
    private String content;

    public Board(String title, String content) {
        this.title = title;
        this.content = content;
    }
}

// Repository
public interface BoardRepository extends JpaRepository<Board, Long> {}
```

## 테스트

```java
@SpringBootTest
class BoardRepositoryTest {

    @Autowired
    private BoardRepository boardRepository;

    static List<Board> boards;

    @BeforeAll
    static void setup() {
        boards = new ArrayList<>();
        for (int i = 0; i < 60000; i++) {
            boards.add(new Board("title", "content"));
        }
    }

    @Test
    @DisplayName("6만 건 saveAll")
    public void addAll(){
        boardRepository.saveAll(boards);
    }

}
```

## 테스트 결과

- 3개의 데이터를 batch insert를 적용한 것과 하지 않았을 때의 로그를 확인
- 예상한 대로 적용 후에는 하나의 쿼리문으로 만들어져 발생하는 것을 볼 수 있음

### 적용 전

![userInsertTest1](/assets/postImages/MysqlBatchInsert/userInsertTest1.png)

### 적용 후

![userInsertTest1](/assets/postImages/MysqlBatchInsert/userInsertTest2.png)

# Identity 전략에서의 Batch Insert

<hr>

- identity 방식에서는 jdbc 레벨에서 일괄 처리 기능을 비활성화시키기에 이전과 같은 방식으로는 불가능
- jdbc batchUpdate를 이용해서 이를 해결함
- batch 사이즈는 이전과 다르게 yml이 아니라 소스 코드 내에서 설정함

## 적용 방법

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?rewriteBatchedStatements=true
```

```java
@Repository
@RequiredArgsConstructor
public class UserJdbcRepository {

    private final JdbcTemplate jdbcTemplate;

    public void saveAll(List<User> users) {
        jdbcTemplate.batchUpdate("insert into users(email, password, name) values(?, ?, ?)"
            , users, 1000, ((ps, argument) -> {
                ps.setString(1, argument.getEmail());
                ps.setString(2, argument.getPassword());
                ps.setString(3, argument.getName());
            }));
    }
}
```

## 준비 내용

```java
// Entity
@Entity
@Table(name = "users")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "user_id")
    Long id;

    String email;
    String password;
    String name;

    public User(String email, String password, String name) {
        this.email = email;
        this.password = password;
        this.name = name;
    }
}

public interface UserRepository extends JpaRepository<User, Long> {}
```

## 테스트

```java
@SpringBootTest
@Transactional
class UserRepositoryTest {

    @Autowired
    private UserJdbcRepository userJdbcRepository;

    static List<User> users;

    @BeforeAll
    static void setup() {
        users = new ArrayList<>();
        for (int i = 0; i < 60000; i++) {
            users.add(new User("email", "password", "name"));
        }
    }

    @Test
    @DisplayName("6만 건 bulk insert")
    public void testBulkInsert() throws Exception {
        userJdbcRepository.saveAll(users);
    }
}
```

## 테스트 결과

- 3개의 데이터를 batch insert를 적용한 것과 하지 않았을 때의 로그를 확인
- 예상한 대로 적용 후에는 하나의 쿼리문으로 만들어져 발생하는 것을 볼 수 있음

### 적용 전

![boardInsertTest1](/assets/postImages/MysqlBatchInsert/boardInsertTest1.png)

### 적용 후

![boardInsertTest1](/assets/postImages/MysqlBatchInsert/boardInsertTest2.png)

# Bach Insert 적용 전/후 속도 비교

<hr>

- 각 6만 건의 데이터를 삽입을 비교
- single insert : 76s
- batch insert : 2s
- 성능 : single insert < batch insert

# Reference

<hr>

- [https://stackoverflow.com/questions/27755461/why-is-hibernate-batching-order-inserts-order-updates-disabled-by-default](https://stackoverflow.com/questions/27755461/why-is-hibernate-batching-order-inserts-order-updates-disabled-by-default)
- [https://jaehun2841.github.io/2020/11/22/2020-11-22-spring-data-jpa-batch-insert/#hibernatejdbcbatch_size](https://jaehun2841.github.io/2020/11/22/2020-11-22-spring-data-jpa-batch-insert/#hibernatejdbcbatch_size)
- [https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#batch-session-batch](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#batch-session-batch)