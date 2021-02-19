---
title: Spring MVC Framework 동작 과정
decription: 평소에 사용하는 Spring Framework가 요청을 받고 뷰를 보여주기까지 어떤 과정을 거치는지에 대해서 알아보자
categories:
 - Spring
tags:
 - Spring
 - MVC
---

> 평소에 사용하는 Spring Framework가 요청을 받고 뷰를 보여주기까지 어떤 과정을 거치는지에 대해서 알아보자

> ## Spring MVC 동작 과정

![springMvcLifeCycle](/assets/springMvcLifeCycle.PNG)

1. `DispacherServlet`이 Client의 요청을 받음
2. `HandlerMapping`이 알맞은 Controller 메소드 정보(핸들러)를 탐색
3. `DispacherServlet`에게 찾은 메소스 정보를 전달
4. `HandlerAdapter`에게 핸들러 호출 위임
5. `HandlerAdapter`는 정보를 토대로 알맞은 Controller를 탐색
6. 로직 처리
7. `DispacherServlet`에게 Model과 View name 전달
8. `View Resolver`는 View name을 통해 해당 View Object를 탐색
9. JSP, Thymeleaf 등을 이용하여 Model Data를 이용하여 View 생성
10. 얻어진 View를 `DispacherServlet`에 전달
11. 최종 결과물 Client에게 전달

> ### Filter와 Interceptor 추가

![springMvcLifeCycle2](/assets/springMvcLifeCycle2.PNG)

- <mark style='background-color: #fff5b1'>Filter는 전체적인 Request단에서 어떤 처리가 필요할 때 동작, 문자 인코딩 등</mark>
  - `DispacherServlet`를 거치기 전 1번 전에 추가된다
- <mark style='background-color: #00FFFF'>Controller 호출 전/후로 필요로 하는 처리가 필요할 때 동작</mark>
  - `Controller` 호출 전/후인 4번 전, 7번 후에 추가된다
