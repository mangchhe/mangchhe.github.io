---
layout: post
title: How to find JVM flag?
categories: java
tags: java
---

```sh
$ java -server -XX:+PrintFlagsFinal  -version

[Global flags]
      int ActiveProcessorCount                     = -1                                        {product} {default}
    uintx AdaptiveSizeDecrementScaleFactor         = 4

...blah blah

openjdk version "17.0.6" 2023-01-17 LTS
OpenJDK Runtime Environment Corretto-17.0.6.10.1 (build 17.0.6+10-LTS)
OpenJDK 64-Bit Server VM Corretto-17.0.6.10.1 (build 17.0.6+10-LTS, mixed mode, sharing)
```

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

```sh
$ java -server -XX:+PrintCommandLineFlags  -version

-XX:ConcGCThreads=2 -XX:G1ConcRefinementThreads=9 -XX:GCDrainStackTargetSize=64 -XX:InitialHeapSize=536870912 -XX:MarkStackSize=4194304 -XX:MaxHeapSize=8589934592 -XX:MinHeapSize=6815736 -XX:+PrintCommandLineFlags -XX:ReservedCodeCacheSize=251658240 -XX:+SegmentedCodeCache -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseG1GC
openjdk version "17.0.6" 2023-01-17 LTS
OpenJDK Runtime Environment Corretto-17.0.6.10.1 (build 17.0.6+10-LTS)
OpenJDK 64-Bit Server VM Corretto-17.0.6.10.1 (build 17.0.6+10-LTS, mixed mode, sharing)
```