---
title: ModelMapper와 MapStruct에 대해서 학습하기
decription: Object Mapping을 위한 ModelMapper와 MapStruct에 대해서 의존성 설정, 매핑 사용법과 차이점을 알아보자
categories:
 - Spring
tags:
 - SpringBoot
 - DTO
 - ModelMapper
 - MapStruct
---

> Object Mapping을 위한 ModelMapper와 MapStruct에 대해서 의존성 설정, 매핑 사용법과 차이점을 알아보자

> ## 개요

`ModelMapper`와 `MapStruct`는 Entity를 DTO로 변환하거나 DTO를 Entity를 변환하려고 할때 사용하게 된다.

**예제 소스 파일** : [Github](https://github.com/mangchhe/WEB_DTO_Tutorial)

> ### Why?

라이브러리를 이용하여 변환하려는 이유가 무엇일까?

``` java
public class Member {
    Name name;
    String phoneNumber;
    int height;
    int weight;
    LocalDateTime createdDate;
  }
```

``` java
public class Name {
    private String firstName;
    private String lastName;
}
```

``` java
public class MemberDTO {
    String firstName;
    String lastName;
    String phoneNumber;
    int height;
    int weight;
    LocalDateTime createdDate;
}
```
다음과 같은 엔티티와 DTO가 존재한다고 할때 `Member -> MemberDTO` 로 변환하기위해서는 아래와 같이 메소드를 대부분 사용할것이라고 생각한다

``` java
public MemberDTO entityToDTO(){
        return MemberDTO.builder()
                .firstName(getName().getFirstName())
                .lastName(getName().getLastName())
                .phoneNumber(getPhoneNumber())
                .height(getHeight())
                .weight(getWeight())
                .createdDate(getCreatedDate())
                .build();
    }
```

이렇게 메소드를 선언하여 직접 사용하면 안좋은 점이 무엇이 있을까?

- 필드의 갯수가 늘어나거나 엔티티간의 연관관계가 늘어날수록 가독성을 떨어질 것이다
- 코드를 작성하는 과정에서 개발자가 실수하여 다른 데이터를 넣을 수 있다
- 반복적인 작업으로 인해 쉽게 피로해질 수 있다
- 필드가 추가나 수정, 삭제가 일어날 경우 변환하는 로직에 대해서 수정이 필요하다

쉽게 말하면, `생산성`과 `유지보수`가 떨어지게 될것이다
이러한 문제를 지금부터 배우려고 하는 라이브러리를 통하여 해결할 수 있다

> ## ModelMapper

> ### 의존성 설정

``` java
dependencies {
  ...
  /* ModelMapper */
  compile 'org.modelmapper:modelmapper:2.1.1'
}
```
build.gradle에 다음과 같이 의존성을 추가하면 된다

다른 빌드 도구 **maven을 사용하는 경우**나 **다른 버전을 사용하고 싶을 경우** [의존성](https://mvnrepository.com/artifact/org.modelmapper/modelmapper)이 경로로 들어가서 의존성 설정을 하면 된다

> ### Entity & DTO

``` java
public class Member {
    Name name;
    String phoneNumber;
    int height;
    int weight;
    LocalDateTime createdDate;
}

public class Name {
    private String firstName;
    private String lastName;
}

public class MemberDTO {
    String firstName;
    String lastName;
    String phoneNumber;
    int height;
    int weight;
    LocalDateTime createdDate;
}
```
해당 코드는 `@Entity`, `@Builder`, `@Data` 가 달려있지 않지만 달려있다고 가정하고 진행하겠다

> ### Bean Configuration

```java
@Configuration
public class ApplicationConfig {

    @Bean
    public ModelMapper modelMapper(){
        ModelMapper modelMapper = new ModelMapper();
        /* modelMapper.getConfiguration().setMatchingStrategy(MatchingStrategies.STRICT); */
        return modelMapper;
    }
}
```
주석이 된 부분은 매칭 전략을 설정하는 부분이며 `Standard`, `Loose`, `Strict` 세 가지가 있고 **Default는 Standard**로 되어있다.

추가적인 내용은 http://modelmapper.org/user-manual/configuration/#matching-strategies 레퍼런스를 참조하길 바란다

> ### Test Code

``` java
@SpringBootTest
class MemberDTOTest {

    @Autowired
    private ModelMapper modelMapper;

    @Test
    public void ModelMapperTest() throws Exception {
        //given
        Member member = Member.builder()
                ...
                .build();
```

아까 Bean으로 등록한 ModelMapper에 DI를 하고 사용하면 되고 Member에 데이터를 삽입했다고 가정하겠다

``` java
//when
MemberDTO result = modelMapper.map(member, MemberDTO.class);
Member result2 = modelMapper.map(result, Member.class);
```

1. `Entity -> DTO`,  `modelMapper.map(Entity 객체, DTO클래스명.class)`
2. `DTO -> Entity`, `modelMapper.map(DTO 객체, Entity클래스명.class)`

사용방법은 위와 같이 변경할 객체와 변경될 객체의 클래스명을 전달하면 변환이 이루어 지는 것을 알 수 있다

``` java
//then

/* Entity -> DTO 변환 확인 */
Assertions.assertThat(result.getFirstName()).isEqualTo(member.getName().getFirstName());
Assertions.assertThat(result.getLastName()).isEqualTo(member.getName().getLastName());
Assertions.assertThat(result.getPhoneNumber()).isEqualTo(member.getPhoneNumber());
Assertions.assertThat(result.getHeight()).isEqualTo(member.getHeight());
Assertions.assertThat(result.getWeight()).isEqualTo(member.getWeight());
Assertions.assertThat(result.getCreatedDate()).isEqualTo(member.getCreatedDate());

/* DTO -> Entity 변환 확인 */
Assertions.assertThat(result2.getName().getFirstName()).isEqualTo(member.getName().getFirstName());
Assertions.assertThat(result2.getName().getLastName()).isEqualTo(member.getName().getLastName());
Assertions.assertThat(result2.getPhoneNumber()).isEqualTo(member.getPhoneNumber());
Assertions.assertThat(result2.getHeight()).isEqualTo(member.getHeight());
Assertions.assertThat(result2.getWeight()).isEqualTo(member.getWeight());
Assertions.assertThat(result2.getCreatedDate()).isEqualTo(member.getCreatedDate());
```

> ### 주의사항

- **만들어지는 대상은 Getter 만드는 대상은 Setter가 필요하다**
Entity가 DTO로 변환된다고 한다면 Entity에는 각 필드값을 읽을 수 있는 Getter가 존재해야되고 DTO는 필드값을 넣을 수 있는 Setter들이 존재해야 한다
- **필드 작명, Standard(Default Staragy) 기준**
필드 이름이 같을 경우 자동으로 매핑이 이루어지지만, 필드이름이 다를 경우 매핑이 이루어지지 않는다
- **연관 관계에 있는 것**
현재 예제를 보면 Memeber가 Name과 연관 관계를 가지고 있다 이때 매핑하기 위해서 이름이 같아도 가능하지만 같은 이름이 존재할 수도 있다 그렇기에 DTO에서 firstName을 nameFirstName 으로 `참조객체이름명+필드명`로 카멜 케이스로 작성해도 된다

> ## MapStruct

> ### 의존성 설정

``` java
dependencies {
...
  /* MapStruct */
  implementation 'org.mapstruct:mapstruct:1.4.1.Final'
  /* Lombok */
  annotationProcessor 'org.projectlombok:lombok'
  /* MapStruct */
  annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.1.Final'
}
```
build.gradle에 다음과 같이 의존성을 추가하면 된다

다른 빌드 도구 `maven을 사용하는 경우` [의존성](https://mapstruct.org/documentation/installation/)이 경로로 들어가서 의존성 설정을 하면 된다

> ### Entity & DTO

``` java
public class Member {
    Name name;
    String phoneNumber;
    int height;
    int weight;
    LocalDateTime createdDate;
}

public class Name {
    private String firstName;
    private String lastName;
}

public class MemberDTO {
    String firstName;
    String lastName;
    String phoneNumber;
    int height;
    int weight;
    LocalDateTime createdDate;
}
```
해당 코드는 `@Entity`, `@Builder`, `@Data` 가 달려있지 않지만 달려있다고 가정하고 진행하겠다

> ### Util Interface

``` java
@Mapper
public interface MemberMapper {
    /* final static */ MemberMapper INSTANCE = Mappers.getMapper(MemberMapper.class);

    @Mapping(target = "name.firstName", expression = "java(memberDTO.getFirstName())")
    @Mapping(target = "name.lastName", expression = "java(memberDTO.getLastName())")
    Member dtoToMember(MemberDTO memberDTO);

    @Mapping(target = "firstName", expression = "java(member.getName().getFirstName())")
    @Mapping(target = "lastName", expression = "java(member.getName().getLastName())")
    MemberDTO entityToMemberDTO(Member member);
}
```
인터페이스에 `@Mapper`를 붙이고 DTO 변환 메소드에 `@Mapping`을 붙여서 적절하게 매핑을 구현하면 된다
target 속성은 변환 되어야 할 필드명이고 expression은 변환 되어지는 객체에서 매핑할 필드을 불러오는 메소드를 불러주면 된다
작성할 때 주의해야 할 점은 `java(메소드)`로 감싸주고 작성해야한다

> ### Test Code

``` java
@Test
public void MapStructTest() throws Exception {
    //given
    Member member = Member.builder()
            ...
            .build();
```
``` java
//when
MemberDTO result = MemberMapper.INSTANCE.entityToMemberDTO(member);
Member result2 = MemberMapper.INSTANCE.dtoToMember(result);
```
아까 만든 mapper 클래스의 정적 변수 INSTANCE를 가져와 매핑 메소드를 호출한다

``` java
//then

/* Entity -> DTO */
Assertions.assertThat(result.getFirstName()).isEqualTo(member.getName().getFirstName());
Assertions.assertThat(result.getLastName()).isEqualTo(member.getName().getLastName());
Assertions.assertThat(result.getPhoneNumber()).isEqualTo(member.getPhoneNumber());
Assertions.assertThat(result.getHeight()).isEqualTo(member.getHeight());
Assertions.assertThat(result.getWeight()).isEqualTo(member.getWeight());
Assertions.assertThat(result.getCreatedDate()).isEqualTo(member.getCreatedDate());

/* DTO -> Entity */
Assertions.assertThat(result2.getName().getFirstName()).isEqualTo(member.getName().getFirstName());
Assertions.assertThat(result2.getName().getLastName()).isEqualTo(member.getName().getLastName());
Assertions.assertThat(result2.getPhoneNumber()).isEqualTo(member.getPhoneNumber());
Assertions.assertThat(result2.getHeight()).isEqualTo(member.getHeight());
Assertions.assertThat(result2.getWeight()).isEqualTo(member.getWeight());
Assertions.assertThat(result2.getCreatedDate()).isEqualTo(member.getCreatedDate());
```

> ### 주의사항

- **만들어지는 대상은 Getter 만드는 대상은 Setter가 필요하다**
Entity가 DTO로 변환된다고 한다면 Entity에는 각 필드값을 읽을 수 있는 Getter가 존재해야되고 DTO는 필드값을 넣을 수 있는 Setter들이 존재해야 한다

- **필드명이 다를 경우나 Default 값을 주고 싶을 경우**

``` java
public class MemberKrDTO {
    String name; /* 이전 필드명 : firstName */
    String sung; /* 이전 필드명 : LastName */
    String phoneNumber;
    int cm; /* 이전 필드명 : height */
    int kg; /* 이전 필드명 : weight */
    int age; /* 추가된 필드명 */
    LocalDateTime createdDate;
}
```
``` java
@Mapping(target = "name", expression = "java(member.getName().getFirstName())")
@Mapping(target = "sung", expression = "java(member.getName().getLastName())")
@Mapping(source = "height", target = "cm")
@Mapping(source = "weight", target = "kg")
@Mapping(target = "age", constant = "25")
MemberKrDTO entityToMemberKrDTO(Member member);
```
source 속성은 변환 할 객체의 필드명이고 target은 변환 되어 질 객체의 필드명을 적어주면 된다
추가로 없는 속성을 전달할 경우 0 또는 null 값이 들어가게 되는데 constant로 값을 넣어주면 default 값으로 설정할 수 있다
``` java
MemberKrDTO result3 = MemberMapper.INSTANCE.entityToMemberKrDTO(member);
```
사용 방법은 위의 같이 해당 변환 메소드를 불러주면 된다

> ## ModelMapper vs MapStruct

> ### ModelMapper와 MapStruct의 속도 비교

``` java
//given
        Member member = Member.builder()
                ...
                .build();
                //when
        long start = System.currentTimeMillis();
        for (int i = 0; i < 500000; i++) {
            modelMapper.map(member, MemberDTO.class);
        }
        long modelMapperDelayTime = System.currentTimeMillis() - start;
        start = System.currentTimeMillis();
        for (int i = 0; i < 500000; i++) {
            MemberMapper.INSTANCE.entityToMemberDTO(member);
        }
        long mapStructDelayTime = System.currentTimeMillis() - start;
        //then
        System.out.println(modelMapperDelayTime / 1000.0);
        System.out.println(mapStructDelayTime / 1000.0);
    }
}
```

![mappingSpeed](/assets/mappingSpeed.PNG)

ModelMapper와 MapStruct를 50만번 동작하여 걸린 시간을 체크한 결과, 필자 컴퓨터 기준으로 ModelMapper는 약 3초 MapStruct는 약 0.008초로 **MapStruct가 월등히 속도가 빠른 걸**로 확인되었다

> ### 속도 차이가 나는 이유

- **ModelMapper**

`modelMapper.map(member, MemberDTO.class)` 매핑이 일어날 때 리플렉션이 발생한다

- **MapStruct**

컴파일 시점에서 어노테이션을 읽어 구현체를 만들어내기 때문에 리플렉션이 발생하지 않는다

![MapStructTest](/assets/MapStructTest.PNG)

각 해당 프로젝트의 빌드 파일에 `@Mapper`를 구현한 폴더에 가게 되면 이렇게 구현체가 생성되어 있는 것을 볼 수 있을 것이다

ModelMapper과 MapStruct에 대해서 더 자세히 알고 싶으시면 해당 Reference를 참고해주세요

- [ModelMapper](http://modelmapper.org/user-manual/)
- [MapStruct](https://mapstruct.org/documentation/reference-guide/)

끝까지 읽어주셔서 감사합니다.
