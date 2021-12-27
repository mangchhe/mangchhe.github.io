---
title: "[Java] JVM(Java Virtual Machine)이란?"
description: 자바 가상머신인 JVM이 무엇인지, 하는 역할과 어떻게 구성되어있는지 알아보자
categories:
 - Java
tags:
 - Java
 - JVM
---

# JVM(Java Virtual Machine)이란?

<hr>

**자바를 실행하기 위한 기계**라고 말할 수 있다 그 말은 자바로 작성된 어플리케이션은 모두 JVM에서만 실행 가능하기 때문에 JVM이 없어서는 안되는 존재이다

## JVM이 해주는 역할?

- Java 어플리케이션을 클래스로더로 읽어들여 자바 API를 함께 실행
- Java와 OS 사이의 중개자 역할, OS에 종속적이지 않고 JVM이 깔려있다면 어디서든 실행 가능
- 메모리 관리, Garbage Collection 수행

## JVM 구성

![JVM](/assets/postImages/JVMConcept/JVM.PNG)

- .java .class
  - 자바 컴파일러를 통하여 .java를 컴파일하여 .class 파일인 바이트코드로 변환

- Class loader
  - Class Loader는 변환시킨 .class 코드를 Runtime Data Area(JVM 메모리 영역)으로 로딩시킴

- Method Area
  - 클래스, 변수, static으로 선언한 변수 정보가 저장되어 있고 모든 스레드 영역에서 공유

- Heap
  - 동적으로 생성된 객체가 저장되는 영역이고, GC(Garbage Collection)의 대상이 되는 공간

- Stack
  - 지역변수나 메소드의 매개변수, 임시적으로 사용되는 변수나 메소드의 정보가 저장되는 공간

![JVM2](/assets/postImages/JVMConcept/JVM2.PNG)

- PC Register
  - 현재 수행중인 JVM 명령어 주소를 저장하는 공간, 스레드가 어떤 부분을 어떤 명령어로 수행할지를 저장하는 공간

- Native Method Stack
  - JAVA가 아닌 다른 언어로 작성된 코드를 위한 공간


![JVM3](/assets/postImages/JVMConcept/JVM3.PNG)

- Execution Engine
  - Class Load에 의해 로드된 클래스 파일의 바이트 코드들을 실행
  - 기계어로 해석한 뒤 JVM 메모리 영역에 배치되어 스레드 동기화나, 가비지 컬렉션을 수행

- Interpreter
  - 명령어를 한줄마다 해석하며 실행

- JIT Compiler
  - 런타임 시간에 한꺼번에 해석하여 실행

- Garbage Collector
  - 메모리 관리 기능을 자동으로 수행한다
  - 애플리케이션이 생성한 객체의 생존 여부를 파악하여 사용되고 있지 않은 객체의 메모리를 해제한다
