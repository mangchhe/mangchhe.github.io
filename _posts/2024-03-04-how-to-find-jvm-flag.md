---
layout: post
title: How to find JVM flag?
categories: java
tags: java
---

`java -server -XX:+PrintFlagsFinal  -version` 명령어는 JVM에 대한 정보(JVM 서버 모드 옵션과 JVM 플래그, Java 버전)를 출력한다.

- -server 옵션: 이 옵션은 JVM을 서버 모드로 실행하도록 지시한다.
- -XX:+PrintFlagsFinal 옵션: 이 옵션은 현재 사용 중인 JVM의 모든 플래그와 그 값을 출력한다.
- -version 옵션: 이 옵션은 현재 사용 중인 Java 버전을 출력한다.

```sh
$ java -server -XX:+PrintFlagsFinal -version

[Global flags]
      int ActiveProcessorCount                     = -1                                        {product} {default}
    uintx AdaptiveSizeDecrementScaleFactor         = 4

...blah blah

openjdk version "17.0.6" 2023-01-17 LTS
OpenJDK Runtime Environment Corretto-17.0.6.10.1 (build 17.0.6+10-LTS)
OpenJDK 64-Bit Server VM Corretto-17.0.6.10.1 (build 17.0.6+10-LTS, mixed mode, sharing)
```

- -XX:+UnlockExperimentalVMOptions: experimental VM 옵션을 해제한다. 출력된 옵션은 향후 릴리스에서 공식적으로 지원될 수 있다.
- -XX:+UnlockDiagnosticVMOptions: dianostic VM 옵션을 해제합니다. 출력된 옵션은 JVM의 동작을 분석하고 진단하는 데 사용될 수 있다.

```sh
$ java -server -XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+PrintFlagsFinal -version

[Global flags]
     bool AbortVMOnCompilationFailure              = false                                  {diagnostic} {default}
    ccstr AbortVMOnException                       =

...blah blah

openjdk version "17.0.6" 2023-01-17 LTS
OpenJDK Runtime Environment Corretto-17.0.6.10.1 (build 17.0.6+10-LTS)
OpenJDK 64-Bit Server VM Corretto-17.0.6.10.1 (build 17.0.6+10-LTS, mixed mode, sharing)
```

- -XX:+PrintCommandLineFlags: JVM이 실행될 때 사용된 커맨드 라인 플래그를 출력한다. 이 옵션을 사용하면 JVM의 실행에 사용된 모든 플래그 값을 확인할 수 있다.

```sh
$ java -server -XX:+PrintCommandLineFlags -version

-XX:ConcGCThreads=2 -XX:G1ConcRefinementThreads=9 -XX:GCDrainStackTargetSize=64 -XX:InitialHeapSize=536870912 -XX:MarkStackSize=4194304 -XX:MaxHeapSize=8589934592 -XX:MinHeapSize=6815736 -XX:+PrintCommandLineFlags -XX:ReservedCodeCacheSize=251658240 -XX:+SegmentedCodeCache -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseG1GC
openjdk version "17.0.6" 2023-01-17 LTS
OpenJDK Runtime Environment Corretto-17.0.6.10.1 (build 17.0.6+10-LTS)
OpenJDK 64-Bit Server VM Corretto-17.0.6.10.1 (build 17.0.6+10-LTS, mixed mode, sharing)
```