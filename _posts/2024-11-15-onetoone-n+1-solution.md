---
layout: post
title: '@OneToOne N+1 Solution'
categories: jpa
tags: jpa
---

> [https://vladmihalcea.com/best-way-onetoone-optional/](https://vladmihalcea.com/best-way-onetoone-optional/)

> **Intellij Warning**
>
> Specifying FetchType. LAZY for the non-owning side of the @OneToOne association will not affect the loading. The related entity will still be loaded as if the FetchType. EAGER is defined. 

`@OneToOne` 연관 관계를 사용할 때, **연관관계의 주인이 아닌 곳**에서 데이터를 조회하면 FetchType.LAZY로 설정했더라도 추가적인 쿼리가 발생할 수 있다. 이는 `LAZY` 설정 시 Hibernate가 프록시 객체를 생성하려고 하지만, **연관 관계의 주인이 아닌 경우** 외래 키(FK)가 없어서 **연관 엔티티가 존재하는지 알 수 없다**. 그렇기 때문에 Hibernate는 추가 쿼리를 실행하게 된다.

해결하기 위한 몇 가지 방법이 있다.

- **optional = false 설정**
  -  연관 엔티티가 항상 존재한다고 가정하기 때문에 프록시 객체를 생성한다.
  -  다만, 무조건 디비에 데이터가 있어야 한다.
  -  `@OneToOne(optional = false)`
- **Fetch Join**
  - 명시적으로 연관 엔티티를 함께 조회해 오기 때문에 추가 쿼리가 필요 없다.
- **엔티티 설계 변경**
  - 디비에서 1:1 관계라 할지라도 엔티티 설계에서는 `@ManyToOne` 으로 표현하면 `LAZY`가 동작한다.