---
title: "[Spring Boot] Fixture Monkey - 테스트 객체 쉽게 만들기"
description: Fixture Monkey를 이용해서 좀 더 가독성 있는 테스트 객체를 만들어보자.
categories:
 - Test
tags:
 - Test
 - SpringBoot
---

# Fixture Monkey란?

<hr>

- 우리는 테스트 객체를 만들기 위해 많은 시간과 노력을 들인다.
- Fixture Monkey를 이용하면 이러한 테스트 객체를 더욱 쉽게 생성할 수 있도록 도와준다.

# 사용하면 좋은 점

<hr>

- 객체를 쉽게 생성할 수 있다. -> 생산성 향상
- 테스트에 있어 중요한 부분만 별도로 세팅할 수 있다. -> 가독성 향상
- 혹시 모를 엣지 케이스 테스트 가능 -> 품질 향상

## 객체를 쉽게 생성할 수 있다.

- 제공되는 함수를 통해 쉽게 테스트 객체를 생성할 수 있다.
- 객체를 생성하기 위해 어떤 값을 넣을지, 어떻게 생성할지 고민할 필요가 없다.

```java
// before
User user = new User(1L, 'username', 'password', '홍길동', 'address', 'state', 'birth', 1000);

// after
User user = fixtureMonkey.giveMeOne(User.class);
```

## 테스트에 있어 중요한 부분만 별도로 세팅할 수 있다.

- 해당 테스트에서 테스트하고자 하는 값을 쉽게 확인이 가능하다.
- `username`과 `password`를 테스트하고자 할 때 fixture-monkey를 사용하면 좀 더 테스트하고자 하는 부분이 명확해진다.

```java
// before
User user = new User(1L, 'username', 'password', '홍길동', 'address', 'state', 'birth', 1000);
or
User user = new User(null, 'username', 'password', null, null, null, null, null);

// after
User user = fixtureMonkey.giveMeBuilder(User.class)
                .set("username", "username")
                .set("password", "password")
                .sample();
```

## 엣지 케이스 테스트 가능

- 억지스러운 예제이지만 만약 두 명의 유저 보유 자산이 1,073,741,824원 이하일 경우
- 이 둘의 보유 자산을 합친 것은 항상 양수라는 것을 검증하려고 한다면 이것은 실패한 케이스이다.
- 왜냐하면 1073741824 + 1073741824가 된다면 overflow가 발생하여 음수가 되기 때문이다.

```java
// 보유 자산이 Size(min=0, max=1073741824)일 경우
User userA = fixtureMonkey.giveMeOne(User.class);
User userB = fixtureMonkey.giveMeOne(User.class);

assertThat(userA.getHoldings() + userB.getHoldings()).isPositive();
```

# 실습

<hr>

- 실습 저장소 → [이동](https://github.com/TIL-Repo/effective-test/tree/main/monkey)
- 이 외 자세한 실습 내용 → [이동](https://naver.github.io/fixture-monkey/kr/)

## dependency

```yml
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    testImplementation 'com.navercorp.fixturemonkey:fixture-monkey-starter:0.3.5'
    ...
}
```

## 예제 클래스

- Fixture Monkey를 이용해서 객체를 생성하여 값을 삽입하기 위해서는 `NoArgsConstructor`, `setter`가 필요하다.

```java
@Data
public class User {

	@NotNull
	private Long id;
	@NotBlank
	private String username;
	@NotBlank
	private String password;
	@NotBlank
	private String name;

	private String address;
	@NotNull
	private UserState state;
	@NotNull
	@Size(min = 8, max = 8)
	private String birth;
	@Size(max = 3)
	private List<@NotBlank @Size(max = 8) String> hobbies;

	private List<String> subjects;

	enum UserState {
		JOIN, DELETE
	}
}
```

## Fixture Monkey 객체 생성

```java
private static final FixtureMonkey fixtureMonkey = FixtureMonkey.create();
```

## 특정 타입 객체 생성

- `giveMeOne(Class<T> type)` : 인자로 생성하고자 하는 타입을 넣어 생성한다.

```java
void start() throws Exception {
    // when
    User user = fixtureMonkey.giveMeOne(User.class);

    // then
    assertAll(
        () -> assertThat(user.getId()).isNotNull(),
        () -> assertThat(user.getUsername()).isNotBlank(),
        () -> assertThat(user.getPassword()).isNotBlank(),
        () -> assertThat(user.getName()).isNotBlank(),
        () -> assertThat(user.getState()).isNotNull(),
        () -> assertThat(user.getBirth()).hasSize(8).isNotNull(),
        () -> assertThat(user.getHobbies()).hasSizeLessThanOrEqualTo(4)
    );
}
```

## 값 세팅

- `giveMeBuilder(Class<T> type).set("key", "value").sample()` : 생성하는 객체에 필드에 값을 고정할 수 있다.

```java
void set() throws Exception {
    // when
    User user = fixtureMonkey.giveMeBuilder(User.class)
        .set("name", "홍길동")
        .set("address", "서울특별시 관악구")
        .set("state", User.UserState.JOIN)
        .sample();

    // then
    assertAll(
        () -> assertThat(user.getId()).isNotNull(),
        () -> assertThat(user.getUsername()).isNotBlank(),
        () -> assertThat(user.getPassword()).isNotBlank(),
        () -> assertThat(user.getName()).isEqualTo("홍길동"),
        () -> assertThat(user.getAddress()).isEqualTo("서울특별시 관악구"),
        () -> assertThat(user.getState()).isEqualTo(User.UserState.JOIN),
        () -> assertThat(user.getBirth()).hasSize(8).isNotNull(),
        () -> assertThat(user.getHobbies()).hasSizeLessThanOrEqualTo(4)
    );
}
```

## ExpressionSpec 클래스

- `ExpressionSpec`을 이용해 객체 설정값들을 저장해놓고 재사용할 수 있다.

```java
void setByExpressionSpec() throws Exception {
    // given
    ExpressionSpec expression = new ExpressionSpec()
        .set("name", "홍길동")
        .set("address", "서울특별시 관악구")
        .set("state", User.UserState.JOIN);
    
    // when
    User user = fixtureMonkey.giveMeBuilder(User.class)
        .set(expression)
        .sample();

    // then
    assertAll(
        () -> assertThat(user.getId()).isNotNull(),
        () -> assertThat(user.getUsername()).isNotBlank(),
        () -> assertThat(user.getPassword()).isNotBlank(),
        () -> assertThat(user.getName()).isEqualTo("홍길동"),
        () -> assertThat(user.getAddress()).isEqualTo("서울특별시 관악구"),
        () -> assertThat(user.getState()).isEqualTo(User.UserState.JOIN),
        () -> assertThat(user.getBirth()).hasSize(8).isNotNull(),
        () -> assertThat(user.getHobbies()).hasSizeLessThanOrEqualTo(4)
    );
}
```

## NULL 처리

- `setNull` : 해당 값이 NULL
- `setNotNull` : 해당 값은 NOT NULL

```java
@Test
void setNull() throws Exception {
    // when
    User user = fixtureMonkey.giveMeBuilder(User.class)
        .setNull("address")
        // .setNotNull("address")
        .sample();

    // then
    assertThat(user.getAddress()).isNull();
    // assertThat(user.getAddress()).isNotNull();
}
```

## Arbitraries 클래스

- Aribitraries 클래스를 이용하여 다양한 값들을 만들어 테스트를 진행할 수 있다.
- 아래 예제는 1부터 100까지 랜덤으로 값을 삽입하여 검증하는 테스트이다.
- [자세한 내용](https://naver.github.io/fixture-monkey/kr/docs/v0.3.x/features/arbitrary/)

```java
void injectRandomValue() throws Exception {
    // when
    User user = fixtureMonkey.giveMeBuilder(User.class)
        .set("id", Arbitraries.longs().between(1, 100))
        .sample();

    // then
    assertThat(user.getId()).isBetween(1L, 100L);
}
```

## 필드 값에 후행 조건 걸기

- `setPostCondition(String expression, Class<U> clazz, Predicate<U> filter)`를 이용해 설정값에 필터링을 할 수 있다.

```java
void setPostConditionByField() throws Exception {
    // when
    User user = fixtureMonkey.giveMeBuilder(User.class)
        .set("id", Arbitraries.longs().between(1, 50))
        .setPostCondition("id", Long.class, it -> it < 10)
        .sample();

    // then
    assertThat(user.getId()).isBetween(1L, 10L);
}
```

# Reference

<hr>

- [https://github.com/naver/fixture-monkey](https://github.com/naver/fixture-monkey)