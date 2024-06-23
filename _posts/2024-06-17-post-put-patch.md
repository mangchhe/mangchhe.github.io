---
layout: post
title: Http Methods (POST, PUT, PATCH)
categories: cs
tags: cs
---

> [https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)

**POST vs PUT**

`POST`는 주로 새로운 리소스를 생성하기 위해 사용하고 `PUT`은 서버에 리소스를 생성하거나 기존 리소스를 대체하기 위해 사용된다.

`PUT`은 멱등성을 가지며 `POST`는 멱등성을 가지고 있지 않다. 그리하여 `PUT`은 여러 번 수행 시 그 결과는 동일하지만 `POST`은 결과가 다를 수 있다. 예를 들어 같은 리소스가 여러 개 생성될 수 있다.

**PUT vs PATCH**

`PUT`은 주로 전체 리소스를 생성하거나 대체하기 위해 사용하고 `PATCH`는 리소스의 일부를 수정하기 위해 사용한다. `PATCH`는 멱등성을 가지고 있지 않다.

| Method | 목적               | Idempotent | 데이터양         |
|--------|------------------|------------|--------------|
| POST   | 새로운 리소스 생성       | No         | 리소스 전체 또는 일부 |
| PUT    | 리소스 전체를 생성하거나 대체 | Yes       | 리소스 전체       |
| PATCH  | 리소스 일부 수정        | No         | 리소스 일부       |