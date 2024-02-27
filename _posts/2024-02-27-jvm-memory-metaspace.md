---
layout: post
title: "JVM Memory: MetaSpace"
categories: java
tags: java
---

Java 8에서 도입된 MetaSpace는 기존의 Permanent Generation (PermGen) 공간을 대체합니다. 클래스 메타데이터를 저장하는 데 사용되며, 애플리케이션의 필요에 따라 동적으로 크기가 조정할 수 있다.

```
Java 7 JVM
<----- Java Heap ----->             <--- Native Memory --->
+------+----+----+-----+-----------+--------+--------------+
| Eden | S0 | S1 | Old | Permanent | C Heap | Thread Stack |
+------+----+----+-----+-----------+--------+--------------+
                        <--------->
                       Permanent Heap

Java 8 JVM
<----- Java Heap -----> <--------- Native Memory --------->
+------+----+----+-----+-----------+--------+--------------+
| Eden | S0 | S1 | Old | Metaspace | C Heap | Thread Stack |
+------+----+----+-----+-----------+--------+--------------+
```

MetaSpace는 JVM 옵션을 통해 초기 크기와 최대 크기 한계를 설정함으로써 관리할 수 있다. 예를 들어, -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m와 같은 옵션을 사용할 수 있다.

```sh
# MaxMetaspaceSize 설정
$ ./gradlew build -Dorg.gradle.jvmargs="-XX:MaxMetaspaceSize=1024m"
$ java -XX:MaxMetaspaceSize=1024m -jar <jar file name>.jar
```

애플리케이션을 운영하다가 `Caused by: java.lang.OutOfMemoryError: Metaspace`와 같은 오류를 만나게 된다면 MetaSpace의 값을 조정해 볼 수 있다.

> [https://www.tutorialspoint.com/what-is-metaspace-in-java](https://www.tutorialspoint.com/what-is-metaspace-in-java)

MetaSpace의 값을 확인하는 방법에 대해서 알아보자.

```sh
$ jps # Java 애플리케이션 JVM 프로세스 ID 확인
67060 app.jar
$ jstat -gc <pid> # GC 사용량, 단위 KB
S0C         S1C         S0U         S1U          EC           EU           OC           OU          MC         MU       CCSC      CCSU     YGC     YGCT     FGC    FGCT     CGC    CGCT       GCT
    0.0      4096.0         0.0      1023.8     106496.0      61440.0      69632.0      37890.5    65344.0    64893.1    8768.0    8558.8     19     0.066     0     0.000     6     0.003     0.069
$ jinfo -flag MaxMetaspaceSize <pid> # JVM 인자 확인, 단위 Byte
-XX:MaxMetaspaceSize=1073741824
```

> **MC (Metaspace Capacity)** : 현재 Metaspace에 할당된 전체 메모리의 용량. 값은 프로그램 실행 중에 필요에 따라 증가할 수 있으며, -XX:MaxMetaspaceSize에 의해 정의된 최대 한도까지 확장될 수 있다.
>
> **MU (Metaspace Utilization)** : 현재 Metaspace에서 실제로 사용 중인 메모리의 양
