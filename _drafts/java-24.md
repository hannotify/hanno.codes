---
layout: post
title: Java 24 TODO - title
date: 18-03-2025 04:30:00 +0200
header:
  teaser: /assets/images/blog/TODO.jpg
excerpt: Java 24 TODO - excerpt
tags: 
- java
---

TODO - write intro
TODO - explain article structure

![TODO](/assets/images/blog/TODO.jpg)
> Image from <a href="https://pxhere.com/en/photo/TODO">PxHere</a>

## Overview

Here's an overview of the JEPs that ship with Java 24, along with their preview status, to which project they belong, what kind of features they add and the things that have changed since Java 23.

| **JEP** | **Title**                                                          |      **Status** |   **Project** | **Feature Type** | **Changes since Java 23** |
| ------: | ------------------------------------------------------------------ | --------------: | ------------: | ---------------: | ------------------------: |
| **404** | Generational Shenandoah                                            |    Experimental |  HotSpot / GC |      Performance |               New feature |
| **450** | Compact Object Headers                                             |    Experimental |       HotSpot |      Performance |               New feature |
| **472** | Prepare to Restrict the Use of JNI                                 |                 |     Core Libs |      Deprecation |               Deprecation |
| **475** | Late Barrier Expansion for G1                                      |                 |  HotSpot / GC |      Performance |               New feature |
| **478** | Key Derivation Function API                                        |         Preview | Security Libs |         Security |               New feature |
| **479** | Remove the Windows 32-bit x86 Port                                 |                 |       HotSpot |      Deprecation |               Deprecation |
| **483** | Ahead-of-Time Class Loading & Linking                              |                 |       HotSpot |      Performance |               New feature |
| **484** | Class-File API                                                     |                 |     Core Libs |   Class-File API |                     Minor |
| **485** | Stream Gatherers                                                   |                 |     Core Libs |          Streams |                      None |
| **486** | Permanently Disable the Security Manager                           |                 | Security Libs |      Deprecation |               Deprecation |
| **487** | Scoped Values                                                      |  Fourth Preview |          Loom |      Concurrency |                     Minor |
| **488** | Primitive Types in Patterns, instanceof and switch                 |  Second Preview |         Amber |         Language |                      None |
| **489** | Vector API                                                         | Ninth Incubator |        Panama |       Vector API |                     Major |
| **490** | ZGC: Remove the Non-Generational Mode                              |                 |  HotSpot / GC |      Deprecation |               Deprecation |
| **491** | Synchronize Virtual Threads Without Pinning                        |                 |       HotSpot |              Fix |               New feature |
| **492** | Flexible Constructor Bodies                                        |   Third Preview |         Amber |      New feature |                      None |
| **493** | Linking Run-Time Images Without JMODs                              |                 | Tools / JLink |      Performance |               New feature |
| **494** | Module Import Declarations                                         |  Second Preview |         Amber |         Language |                     Minor |
| **495** | Simple Source Files and Instance Main Methods                      |  Fourth Preview |         Amber |         Language |         Name changes only |
| **496** | Quantum-Resistant Module-Lattice-Based Key Encapsulation Mechanism |                 | Security Libs |         Security |               New feature |
| **497** | Quantum-Resistant Module-Lattice-Based Digital Signature Algorithm |                 | Security Libs |         Security |               New feature |
| **498** | Warn upon Use of Memory-Access Methods in sun.misc.Unsafe          |                 |     Core Libs |      Deprecation |               Deprecation |
| **499** | Structured Concurrency                                             |  Fourth Preview |          Loom |      Concurrency |                      None |
| **501** | Deprecate the 32-bit x86 Port for Removal                          |                 |       HotSpot |      Deprecation |               Deprecation |

## New features

Let's start with the most exciting part: the brand-new features that made it to Java 24.

### JEP 404: Generational Shenandoah (Experimental)

TODO

#### More Information

For more information on this feature, see [JEP TODO](https://openjdk.org/jeps/todo).

### JEP 450: Compact Object Headers (Experimental)

TODO

#### More Information

For more information on this feature, see [JEP TODO](https://openjdk.org/jeps/todo).

### JEP 475: Late Barrier Expansion for G1

TODO

#### More Information

For more information on this feature, see [JEP TODO](https://openjdk.org/jeps/todo).

### JEP 478: Key Derivation Function API (Preview)

TODO

#### More Information

For more information on this feature, see [JEP TODO](https://openjdk.org/jeps/todo).

### JEP 483: Ahead-of-Time Class Loading & Linking

TODO

#### More Information

For more information on this feature, see [JEP TODO](https://openjdk.org/jeps/todo).

### JEP 491: Synchronize Virtual Threads Without Pinning

TODO

#### More Information

For more information on this feature, see [JEP TODO](https://openjdk.org/jeps/todo).

### JEP 493: Linking Run-Time Images Without JMODs

TODO

#### More Information

For more information on this feature, see [JEP TODO](https://openjdk.org/jeps/todo).

### JEP 496: Quantum-Resistant Module-Lattice-Based Key Encapsulation Mechanism

TODO

#### More Information

For more information on this feature, see [JEP TODO](https://openjdk.org/jeps/todo).

### JEP 497: Quantum-Resistant Module-Lattice-Based Digital Signature Algorithm

TODO

#### More Information

For more information on this feature, see [JEP TODO](https://openjdk.org/jeps/todo).

## Repreviews and finalizations

Now it's time to take a look at a few features that might already be familiar to you, because they were introduced in a previous version of Java. They have been repreviewed (or finalized) in Java 24, with only minor changes compared to Java 23 in most cases. Therefore, to avoid a very lengthy article, we'll outline these changes and link to a previous article for a full feature description, should you wish to refresh your memory.

### JEP 484: Class-File API

TODO

#### What's Different From Java 23?

TODO

#### More Information

For more information on this feature, see [JEP TODO](https://openjdk.org/jeps/todo) or the [full feature description](/2024/09/17/java-23-has-arrived/#TODO) that was published earlier.

### JEP 485: Stream Gatherers

TODO

#### What's Different From Java 23?

TODO

#### More Information

For more information on this feature, see [JEP TODO](https://openjdk.org/jeps/todo) or the [full feature description](/2024/09/17/java-23-has-arrived/#TODO) that was published earlier.

### JEP 487: Scoped Values (Fourth Preview)

*Scoped values* enable the sharing of immutable data within and across threads. They are preferred to thread-local variables, especially when using large numbers of virtual threads.

#### What's Different From Java 23?

TODO

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 487](https://openjdk.org/jeps/487) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-481-scoped-values-third-preview) that was published earlier.

### JEP 488: Primitive Types in Patterns, instanceof and switch (Second Preview)

TODO

#### What's Different From Java 23?

TODO

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 488](https://openjdk.org/jeps/488) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-455-primitive-types-in-patterns-instanceof-and-switch-preview) that was published earlier.

### JEP 489: Vector API (Ninth Incubator)

TODO

#### What's Different From Java 23?

TODO

The Vector API will keep incubating until necessary features of Project Valhalla become available as preview features. When that happens, the Vector API will be adapted to use them, and it will be promoted from incubation to preview.

#### More Information

For more information on this feature, see [JEP 489](https://openjdk.org/jeps/489) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-469-vector-api-eighth-incubator) that was published earlier.

### JEP 492: Flexible Constructor Bodies (Third Preview)

TODO

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### What's Different From Java 23?

TODO

#### More Information

For more information on this feature, see [JEP 492](https://openjdk.org/jeps/492) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-482-flexible-constructor-bodies-second-preview) that was published earlier.

### JEP 494: Module Import Declarations (Second Preview)

TODO

#### What's Different From Java 23?

TODO

#### More Information

For more information on this feature, see [JEP 494](https://openjdk.org/jeps/494) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-476-module-import-declarations-preview) that was published earlier.

### JEP 495: Simple Source Files and Instance Main Methods (Fourth Preview)

TODO

#### What's Different From Java 23?

TODO

#### More Information

For more information on this feature, see [JEP 495](https://openjdk.org/jeps/495) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-477-implicitly-declared-classes-and-instance-main-methods-third-preview) that was published earlier.

### JEP 499: Structured Concurrency (Fourth Preview)

TODO

#### What's Different From Java 23?

Nothing was changed, the API is re-previewed in Java 24 to give more time for feedback from real world usage.

TODO: add remark on any upcoming API changes

#### More Information

For more information on this feature, see [JEP 499](https://openjdk.org/jeps/499) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-480-structured-concurrency-third-preview) that was published earlier.

## Deprecations 

Java 24 also deprecates a few older features that weren't used that much. Let's see which ones were involved in this cleanup.

### JEP 472: Prepare to Restrict the Use of JNI

TODO

#### More Information

For more information on this feature, see [JEP 472](https://openjdk.org/jeps/472).

### JEP 479: Remove the Windows 32-bit x86 Port

TODO

#### More Information

For more information on this feature, see [JEP 479](https://openjdk.org/jeps/479).

### JEP 486: Permanently Disable the Security Manager

TODO

#### More Information

For more information on this feature, see [JEP 486](https://openjdk.org/jeps/486).

### JEP 490: ZGC: Remove the Non-Generational Mode

TODO

#### More Information

For more information on this feature, see [JEP 490](https://openjdk.org/jeps/490).

### JEP 498: Warn upon Use of Memory-Access Methods in sun.misc.Unsafe

TODO

#### More Information

For more information on this feature, see [JEP 498](https://openjdk.org/jeps/498).

### JEP 501: Deprecate the 32-bit x86 Port for Removal

TODO

#### More Information

For more information on this feature, see [JEP 501](https://openjdk.org/jeps/501).

## Final thoughts

TODO - write outro that summarizes the article
