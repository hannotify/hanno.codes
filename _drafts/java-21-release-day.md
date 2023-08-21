---
layout: post
title: Java 21 is Available Today! Here's What's Inside
date: 19-09-2023 04:30:00 +0200
header:
  teaser: TODO
tags: 
- java
---

Today's the first day of Java 21's availability! It's been six months since Java 20 was released, and so it's time for another fresh wave of Java features. This post will take you on a tour through all JEPs that are associated with this release and give you a brief introduction to each one of them. Where applicable the differences with Java 20 are highlighted and a few typical use cases are provided, so that you'll be more than ready to use these features after you've finished reading!

![TODO](/assets/images/blog/TODO.jpg)
> Image from <a href="https://TODO">TODO</a>

## From Project Amber

Java 21 contains five features that originated from [Project Amber](https://openjdk.org/projects/amber/):

* String Templates;
* Record Patterns;
* Pattern Matching for switch;
* Unnamed Patterns and Variables;
* Unnamed Classes and Instance Main Methods.

> The goal of Project Amber is to explore and incubate smaller, productivity-oriented Java language features.

### JEP 430: String Templates (Preview)

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 430](https://openjdk.org/jeps/430).

### JEP 440: Record Patterns

TODO

#### What's Different From Java 20?

Apart from some minor editorial changes, the main change since the second preview is to remove support for record patterns appearing in the header of an enhanced for statement. This feature may be re-proposed in a future JEP.

TODO: why?

#### More Information

For more information on this feature, see [JEP 440](https://openjdk.org/jeps/440).

### JEP 441: Pattern Matching for switch

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 441](https://openjdk.org/jeps/441).

### JEP 443: Unnamed Patterns and Variables (Preview)

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 443](https://openjdk.org/jeps/443).

### JEP 445: Unnamed Classes and Instance Main Methods (Preview)

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 445](https://openjdk.org/jeps/445).

## From Project Loom

Java 20 contains three features that originated from [Project Loom](http://openjdk.java.net/projects/loom/):

* Virtual Threads;
* Scoped Values;
* Structured Concurrency.

> Project Loom strives to simplify maintaining concurrent applications in Java by introducing *virtual threads* and an API for *structured concurrency*, among other things.

### JEP 444: Virtual Threads

Threads have been a part of Java since the very beginning, and since the start of Project Loom we've gradually started calling them 'platform threads' instead. A platform thread runs Java code on an underlying OS thread and captures the OS thread for the code's entire lifetime. The number of platform threads is therefore limited to the number of available OS threads.

Modern applications, however, might need many more threads than that; when dealing with tens of thousands of requests at the same time, for example. This is where *virtual threads* come in. A virtual thread is an instance of `java.lang.Thread` that runs Java code on an underlying OS thread, but does not capture the OS thread for the code's entire lifetime. This means that many virtual threads can run their Java code on the same OS thread, effectively sharing it. The number of virtual threads can thus be much larger than the number of available OS threads.

Aside from being plentiful, virtual threads are also cheap to create and dispose of. This means that a web framework, for example, can dedicate a new virtual thread to the task of handling a request and still be able to process thousands or even millions of requests at once.

#### Typical Use Cases

Using virtual threads does not require learning new concepts, though it may require unlearning habits developed to cope with today's high cost of threads. Virtual threads will not only help application developers; they will also help framework designers provide easy-to-use APIs that are compatible with the platform's design without compromising on scalability.

#### Creating Virtual Threads

Just like a platform thread, a virtual thread is an instance of `java.lang.Thread`. So you can use a virtual thread in exactly the same way as a platform thread.

Creating a virtual thread is a bit different, but just as easy as creating a platform thread:

```java
var platformThread = new Thread(() -> {
    // do some work in a platform thread
});
platformThread.start();

var virtualThread = Thread.startVirtualThread(() -> {
    // do some work in a virtual thread
});
virtualThread.start();
```

When your code uses the `ExecutorService` interface already, switching to virtual threads will take even less effort:

```java
var platformThreadsExecutor = Executors.newCachedThreadPool();
platformThreadsExecutor.submit(() -> {
    // do some work in a platform thread
});
platformThreadsExecutor.close();

try (var virtualThreadsExecutor = Executors.newVirtualThreadPerTaskExecutor()) {
    virtualThreadsExecutor.submit(() -> {
        // do some work in a virtual thread
    });
} // close() is called implicitly
```

Note that the `ExecutorService` interface was adjusted in Java 19 to extend `AutoCloseable`, so it can now be used in a try-with-resources construct.

#### What's Different From Java 20?

Based on developer feedback the following changes were made to virtual threads compared to Java 20:

* Virtual threads now always support thread-local variables. Guaranteed support for thread-local variables ensures that many more existing libraries can be used unchanged with virtual threads, and helps with the migration of task-oriented code to use virtual threads.
* Virtual threads created directly with the Thread.Builder API (as opposed to those created through `Executors.newVirtualThreadPerTaskExecutor()`) are now also, by default, monitored throughout their lifetime and observable via the new thread dump mechanism that also was introduced with the virtual threads feature.

#### More Information

For more information on this feature, see [JEP 444](https://openjdk.org/jeps/444).

### JEP 446: Scoped Values (Preview)

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 446](https://openjdk.org/jeps/446).

### JEP 453: Structured Concurrency (Preview)

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 453](https://openjdk.org/jeps/453).

## From Project Panama

Java 21 contains two features that originated from [Project Panama](http://openjdk.java.net/projects/panama/):

* Foreign Function & Memory API;
* Vector API.

> Project Panama aims to improve the connection between the JVM and foreign (non-Java) libraries.

### JEP 442: Foreign Function & Memory API (Third Preview)

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 442](https://openjdk.org/jeps/442).

### JEP 448: Vector API (Sixth Incubator)

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 448](https://openjdk.org/jeps/448).

## Hotspot

TODO

* Generational ZGC;
* Deprecate the Windows 32-bit x86 Port for Removal;
* Prepare to Disallow the Dynamic Loading of Agents.

### JEP 439: Generational ZGC

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 439](https://openjdk.org/jeps/439).

### JEP 449: Deprecate the Windows 32-bit x86 Port for Removal

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 449](https://openjdk.org/jeps/449).

### JEP 451: Prepare to Disallow the Dynamic Loading of Agents

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 451](https://openjdk.org/jeps/451).

## Other Libs

TODO

* Sequenced Collections
* Key Encapsulation Mechanism API

### JEP 431: Sequenced Collections

TODO
(from Core Libs)

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 431](https://openjdk.org/jeps/431).

### JEP 452: Key Encapsulation Mechanism API

TODO
(from Security Libs)

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 452](https://openjdk.org/jeps/452).

## Final thoughts

TODO
