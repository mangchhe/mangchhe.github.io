---
title: 다대일 양방향 연관관계 주인 설정에 대해서 학습하기
decription: 다대일 양방향 연관관계의 주인을 어떻게 설정하는지 사용할 때 어떤 부분에 대해서 조심해야되는지에 대해서 배워보자
categories:
 - JPA
tags:
 - SpringBoot
 - Spring Data JPA
 - DAO
---

> 다대일 양방향 관계에서 연관관계의 주인을 어떻게 설정하는지 사용할 때 어떤 부분에 대해서 조심해야되는지에 대해서 배워보자

## 개요

![mappedby](/assets/mappedby.png)

위 그림과 같이 게시글(Board)과 댓글(Comment)의 다대일 양방향 연관관계에 대해서 살펴보겠습니다.

**예제 소스 파일** : [Github](https://github.com/mangchhe/WEB_JPA_Tutorial)

**참고** : 자바 ORM 표준 JPA 프로그래밍, 김영한 저자

## 연관관계의 주인을 설정하는 이유

`@OneToMany`을 설정했는데 왜 `mappedBy`라는 속성을 이용하여 연관관계 주인을 정해주어야 할까?

위에 그림을 보면 아래와 같이 되어있다

1. Board가 Comment를 참조
2. Comment가 Board를 참조

이 둘을 서로 양방향 연관관계라고 하지만 객체는 양방향 연관관계라는 것이 존재하지 않고 **서로 다른 두 객체가 단방향으로 참조하고 있는 것**이다 즉, **두개가 서로 다른 객체를 가르키고 있는 것**이다

서로 다른 두 객체를 관리하기에는 부담이 두배가 되기 때문에 둘 중 하나를 외래키, 즉 연관관계 주인으로 정해야 할것이다

## 연관관계의 주인 설정하는 방법

``` java
@Entity
@Getter @Setter
public class Comment {

    ...

    @ManyToOne
    @JoinColumn(name = "board_id")
    Board board;
}
```
``` java
@Entity
@Getter @Setter
public class Board {

    ...

    @OneToMany(mappedBy = "board")
    List<Comment> comments = new ArrayList<>();
}
```

그림과 같이 연관관계를 구성하려면 위 소스와 같이 작성하게 된다

`ManyToOne`과 같이 **외래키를 갖는 곳을 연관관계의 주인**으로 정하고 `OneToMany`와 같이 주인이 아닌 곳에 `mappedBy` 속성으로 외래키에 준 필드명을 넣어주면 된다

> 이때 Entity에 @Setter 사용은 지양하는 것이 왠만하면 변경에 있어서 의미있는 이름을 갖는 사용자 메소드를 선언하여 사용하는 것이 좋다 편의상 나는 @Setter 어노테이션을 이용하여 진행하겠습니다

## 사용 시 주의사항

### 문제 발생

``` java
public class Board {

    ...

    @OneToMany(mappedBy = "board")
    List<Comment> comments = new ArrayList<>();

    public void addComment(Comment comment){
        comments.add(comment);
    }
}
```

Board 엔티티에 다음과 같이 게시글에 댓글을 추가하기 위해 addComment 라는 메소드를 하나 정의했다고 생각하고 테스트케이스를 작성해보겠습니다

``` java
@SpringBootTest
class BoardTest {

    @Autowired
    private BoardRepository boardRepository;
    @Autowired
    private CommentRepository commentRepository;

    @Test
    public void mappedby() throws Exception {
        // given
        Board board = new Board();
        board.setContent("게시글 내용");
        boardRepository.save(board);

        Comment comment = new Comment();
        comment.setContent("덧글 내용");
        // when
        board.addComment(comment);
        commentRepository.save(comment);
    }
}
```

![mappedby_board](/assets/mappedby_board.PNG)
![mappedby_comment](/assets/mappedby_comment.PNG)

결과는 다음과 같다 분명 `board.addComment(comment)` 를 이용하여 댓글을 저장하였지만 데이터베이스를 열어보면 저장되어있지 않을 것을 볼 수 있다

이유는 연관관계의 주인이 아닌 곳에서 추가하려고 했기 외래키를 조작하려고 했기 때문이다

### 문제 해결

**연관관계의 주인만이 외래키를 관리할 수 있고 연관관계의 주인이 아니면 읽기만 할 수 있다**

그럼 문제를 해결하러 가보자

``` java
public class Board {

    ...

    @OneToMany(mappedBy = "board")
    List<Comment> comments = new ArrayList<>();

    public void addComment(Comment comment){
        comment.setBoard(this); // 추가
        comments.add(comment);
    }
}
```

`comment.setBoard(this)` 해당 메소드를 불러올 때 해당 주인을 가지고 외래키 설정을 해주면 된다

![mappedby_board2](/assets/mappedby_board2.PNG)
![mappedby_comment2](/assets/mappedby_comment2.PNG)

데이터베이스를 열어 확인해보면 제대로 적용이 된것을 확인할 수 있을 것이다

### 의문점

``` java
public class Board {

    ...

    @OneToMany(mappedBy = "board")
    List<Comment> comments = new ArrayList<>();

    public void addComment(Comment comment){
        comment.setBoard(this);
        comments.add(comment); // <---
    }
}
```

왜 굳이 연관관계의 주인이 아닌 상대에게도 값을 저장해야되나? 데이터베이스에 저장되는 것도 아닌데라는 의문이 생길 것이다

`comments.add(comment)` 해당 메소드가 필요없고 또한 `addComment` 메소드 자체가 필요없고 그냥 comment 객체(댓글)을 생성할 때 바로 게시글을 저장하면 되지 않을까 할 될것이다

우리는 현재 ORM(Object-Relational Mapping) 기술 표준인 JPA를 사용하고 있다 즉, 객체를 통해서 데이터베이스 데이터를 다루고 있다는 뜻이고 **객체지향적 관점으로 보게 되면 양쪽 다 데이터를 가지고 있어야 한다고 봐야 될 것**이다

또는 **JPA를 사용하지 않고 그냥 순수 객체를 이용하여 데이터를 사용하려고 할때** 문제가 발생할 수 도 있기 때문에 사전에 방지할 수 있다

끝까지 읽어주셔서 감사합니다.
