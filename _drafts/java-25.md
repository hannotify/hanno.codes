---
layout: post
title: TODO
date: 16-09-2025 04:30:00 +0200
header:
  teaser: /assets/images/blog/todo.jpg
excerpt: TODO
tags: 
- java
---

Java 25 ... TODO

![TODO](/assets/images/blog/todo.jpg)
> Photo by TODO, from <a href="https://www.pexels.com/photo/todo/">TODO</a>

## JEP Overview

To start off, let's look at an overview of the JEPs that ship with Java 25. This table contains the preview status for all JEP's, to which project they belong, what kind of features they add and the things that have changed since Java 24.

TODO

## New features

Let's start with the JEP's that add brand-new features to Java 25.

### Core Libs

Java 25 contains a single new feature that is part of the Core Libs:

* Stable Values (Preview)

#### JEP 502: Stable Values (Preview)

TODO

##### More Information

For more information on this feature, read [JEP 502](https://openjdk.org/jeps/502).

### HotSpot

Java 25 introduces two new features in [HotSpot](https://openjdk.org/groups/hotspot/):

* Ahead-of-Time Command-Line Ergonomics
* Ahead-of-Time Method Profiling

> The HotSpot JVM is the runtime engine that is developed by Oracle. It translates Java bytecode into machine code for the host operating system's processor architecture.

#### JEP 514: Ahead-of-Time Command-Line Ergonomics

TODO

##### More Information

For more information on this feature, read [JEP 514](https://openjdk.org/jeps/514).

#### JEP 515: Ahead-of-Time Method Profiling

TODO

##### More Information

For more information on this feature, read [JEP 515](https://openjdk.org/jeps/515).

### Security Libs

Java 25 introduces a single new feature that is part of the Security Libs:

* PEM Encodings of Cryptographic Objects (Preview)

#### JEP 470: PEM Encodings of Cryptographic Objects (Preview)

TODO

##### More Information

For more information on this feature, read [JEP 470](https://openjdk.org/jeps/470).

### Java Flight Recorder

Java 25 introduces three new features that are part of the Java Flight Recorder:

* JFR CPU-Time Profiling (Experimental)
* JFR Cooperative Sampling
* JFR Method Timing & Tracing

#### JEP 509: JFR CPU-Time Profiling (Experimental)

TODO

##### More Information

For more information on this feature, read [JEP 509](https://openjdk.org/jeps/509).

#### JEP 518: JFR Cooperative Sampling

TODO

##### More Information

For more information on this feature, read [JEP 518](https://openjdk.org/jeps/518).

#### JEP 520: JFR Method Timing & Tracing

TODO

##### More Information

For more information on this feature, read [JEP 520](https://openjdk.org/jeps/520).

## Repreviews and Finalizations

Now it's time to take a look at a few features that may already be familiar to you, because they were introduced in a previous version of Java. They have been repreviewed (or finalized) in Java 25, with only minor changes compared to Java 24 in most cases. Therefore, to avoid a very lengthy article, we'll outline these changes and link to a previous article for a full feature description, should you wish to refresh your memory.

### JEP 505: Structured Concurrency (Fifth Preview)

TODO

#### What's Different From Java 24?

TODO

#### More Information

For more information on this feature, read [JEP 505](https://openjdk.org/jeps/505) or the [full feature description](https://hanno.codes/todo) from a previous article.

### JEP 506: Scoped Values

TODO

#### What's Different From Java 24?

TODO

#### More Information

For more information on this feature, read [JEP 506](https://openjdk.org/jeps/506) or the [full feature description](https://hanno.codes/todo) from a previous article.

### JEP 507: Primitive Types in Patterns, instanceof, and switch (Third Preview)

TODO

#### What's Different From Java 24?

TODO

#### More Information

For more information on this feature, read [JEP 507](https://openjdk.org/jeps/507) or the [full feature description](https://hanno.codes/todo) from a previous article.

### JEP 508: Vector API (Tenth Incubator)

TODO

#### What's Different From Java 24?

TODO

#### More Information

For more information on this feature, read [JEP 508](https://openjdk.org/jeps/508) or the [full feature description](https://hanno.codes/todo) from a previous article.

### JEP 510: Key Derivation Function API

TODO

#### What's Different From Java 24?

TODO

#### More Information

For more information on this feature, read [JEP 510](https://openjdk.org/jeps/510) or the [full feature description](https://hanno.codes/todo) from a previous article.

### JEP 511: Module Import Declarations

TODO

#### What's Different From Java 24?

TODO

#### More Information

For more information on this feature, read [JEP 511](https://openjdk.org/jeps/511) or the [full feature description](https://hanno.codes/todo) from a previous article.

### JEP 512: Compact Source Files and Instance Main Methods

TODO

#### What's Different From Java 24?

TODO

#### More Information

For more information on this feature, read [JEP 512](https://openjdk.org/jeps/512) or the [full feature description](https://hanno.codes/todo) from a previous article.

### JEP 513: Flexible Constructor Bodies

TODO

#### What's Different From Java 24?

TODO

#### More Information

For more information on this feature, read [JEP 513](https://openjdk.org/jeps/513) or the [full feature description](https://hanno.codes/todo) from a previous article.

### JEP 519: Compact Object Headers

JDK 24 introduced compact object headers as an experimental feature, which enabled a reduction of the object header size to 64 bits. Since then, compact object headers have proven their stability and performance. They have been tested at Oracle by running the full JDK test suite. They have also been tested at Amazon by hundreds of services in production, most of them using backports of the feature to JDK 21 and JDK 17. On top of that, various other experiments have demonstrated that enabling compact object headers improves performance.

#### What's Different From Java 24?

The experimental status has been dropped, which means compact object headers have now become a product feature. They can be enabled via the command-line options:

```bash
$ java -XX:+UseCompactObjectHeaders ...
```

This means the `-XX:+UnlockExperimentalVMOptions` option that was required in Java 24 is no longer necessary. In later releases, we can expect the feature to become enabled by default and eventually the code for legacy object headers will be removed altogether.

#### More Information

For more information on this feature, including references to the conducted experiments that proved the better performance, read [JEP 519](https://openjdk.org/jeps/519) or the [full feature description](https://hanno.codes/2025/03/18/java-24-rolls-out-today/#jep-450-compact-object-headers-experimental) from a previous article.

### JEP 521: Generational Shenandoah

The Shenandoah garbage collector is an ultra-low pause time garbage collector. It has been [available for production use since Java 15](https://openjdk.org/jeps/379) and has been designed to dramatically reduce garbage collection pause times, regardless of the heap size that is used. It can achieve these low pause times because most of the work is done before the GC pause, in a series of preparation steps. Shenandoah marks and compacts any heap objects eligible for garbage collection, while regular Java user threads are still running.

Java 24 introduced an experimental extension to Shenandoah that maintains separate generations for young and old objects, allowing Shenandoah to collect young objects more frequently. This results in a significant performance gain for applications running with generational Shenandoah, without sacrificing any of the valuable properties that the garbage collector is already known for.

The reason for handling young and old objects separately stems from the [weak generational hypothesis](https://docs.oracle.com/en/java/javase/17/gctuning/garbage-collector-implementation.html#GUID-71D796B3-CBAB-4D80-B5C3-2620E45F6E5D), which states that young objects tend to die young, while old objects tend to stick around. This means that collecting young objects requires fewer resources and yields more memory, while collecting old objects requires more resources and yields less memory. This is the reason we can improve the performance of applications that use Shenandoah by collecting young objects more frequently.

#### What's Different From Java 24?

The experimental status has now been dropped, which means generational mode has now become a product feature. Compared to JDK 24, many stability and performance improvements have been implemented, and extensive testing on multiple platforms has been performed.

To run your workload with generational Shenandoah in Java 25, the following configuration is needed:

```bash
$ java ... -XX:+UseShenandoahGC -XX:ShenandoahGCMode=generational
```

This means the `-XX:+UnlockExperimentalVMOptions` option that was required in Java 24 is no longer necessary. Note that no changes were made to Shenandoah's default behaviour. This may still change in a future release, though.

#### More Information

For more information on this feature, read [JEP 521](https://openjdk.org/jeps/521) or the [full feature description](https://hanno.codes/todo) from a previous article.

## Deprecations, Removals & Restrictions

Java 25 also deprecates a few older features that weren't used that much and restricts a few other features that came with certain risks. Let's see which ones were involved in this effort to improve stability.

### JEP 503: Remove the 32-bit x86 Port

This JEP removes the 32-bit x86 (Linux) port, which was to be expected after [its deprecation in Java 24](https://hanno.codes/2025/03/18/java-24-rolls-out-today/#jep-501-deprecate-the-32-bit-x86-port-for-removal). The affected users are expected to already have migrated to 64-bit JVMs.

Supporting multiple platforms has been the focus of the Java ecosystem since the beginning. But older platforms cannot be supported indefinitely—the effort that was required to maintain this port exceeded its advantages. Keeping it up-to-date with new features like Loom, the Foreign Function & Memory API (FFM), the Vector API, and late GC barrier expansion represented a significant cost. So it's time to say goodbye to this port!

#### More Information

For more information on this removal, read [JEP 503](https://openjdk.org/jeps/503).

## Final thoughts

TODO