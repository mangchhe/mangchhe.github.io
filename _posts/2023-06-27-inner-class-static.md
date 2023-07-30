---
layout: post
title: Why does a memory leak occur from an inner class in Java?
categories: java
tags: java
---

정적 멤버 클래스가 아닌 비정적 멤버 클래스를 사용하면 의도치 않은 메모리 누수를 발생시킨다. **멤버 클래스는 바깥 인스턴스에 접근할 일이 없다면 반드시 static을 붙여 정적 멤버 클래스로 만들어야 한다.**

```java
class OuterClass {
    ...
    class InnerClass {
        ...
    }
    statc class StaticNestedClass {

    }
}
```

>
An instance of InnerClass can exist only within an instance of OuterClass and has direct access to the methods and fields of its enclosing instance.
To instantiate an inner class, you must first instantiate the outer class. Then, create the inner object within the outer object with this syntax:

비정적 멤버 클래스는 외부 클래스의 인스턴스 내에서만 존재할 수 있으며 해당 인스턴스의 필드나 메서드에 직접 접근할 수 있다.
비정적 멤버 클래스를 인스턴스화하기 위해서는 바깥 클래스를 먼저 인스턴스화해야 한다고 한다.

```java
OuterClass outerObject = new OuterClass();
OuterClass.InnerClass innerObject = outerObject.new InnerClass();
```

정적 멤버 클래스는 바깥 클래스를 인스턴스화 필요 없이 바로 생성이 가능하다.

```java
StaticNestedClass staticNestedObject = new StaticNestedClass();
```

바이트 코드로 변환된 코드를 보게 되면 비정적 멤버 클래스는 바깥 클래스를 인자로 받아 사용하는 것을 알 수 있다.

```java
new OuterClass.InnerClass(new OuterClass());
new OuterClass.StaticNestedClass();
```

이렇듯 비정적 멤버 클래스는 외부 클래스에 대해서 암시적 참조를 가지고 있고 GC에 의해 수거되지 않아 메모리 누수를 발생시킨다.

### References

- [https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html)
- Effective Java 3/E, Item 24