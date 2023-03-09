---
layout: post
title: It's Java 20 Release Day! Here's What's New
date: 21-03-2023
header:
  teaser:
tags: 
- java
---

Hurray, today is Java 20 release day! 

TODO

## From Project Amber

The goal of [Project Amber](https://openjdk.org/projects/amber/) is to explore and incubate smaller, productivity-oriented Java language features. Java 20 contains two JEPS that originate from Project Amber:

* Record Patterns;
* Pattern Matching for switch.

### JEP 432: Record Patterns (Second Preview)

[feature summary]

#### What's Different From Java 19?

[changes since previous preview]

#### More Information

For more information on this feature, see [JEP 432](https://openjdk.org/jeps/432).

### JEP 433: Pattern Matching for switch (Fourth Preview)

The feature 'Pattern Matching for switch' that was first introduced in Java 16 has reached its fourth preview status now that Java 20 has been released.
...

[feature summary]

#### What's Different From Java 19?

[changes since previous preview]

#### More Information

For more information on this feature, see [JEP 433](https://openjdk.org/jeps/433).

## From Project Loom

[Project Loom](http://openjdk.java.net/projects/loom/) strives to simplify maintaining concurrent applications in Java by introducing *virtual threads* and an API for *structured concurrency*, among other things. Java 20 contains three JEPs that originate from Project Loom: 

* Virtual Threads;
* Scoped Values;
* Structured Concurrency.

### JEP 436: Virtual Threads (Second Preview)

[feature summary]

#### What's Different From Java 19?

[changes since previous preview]

#### More Information

For more information on this feature, see [JEP 436](https://openjdk.org/jeps/436).

### JEP 429: Scoped Values (Incubator)

*Scoped values* enable the sharing of immutable data within and across threads.
They are preferred to thread-local variables, especially when using large numbers of virtual threads.

#### ThreadLocal

Since Java 1.2 we can make use of `ThreadLocal` variables, which confine a certain value to the thread that created it.
In [some cases](https://stackoverflow.com/a/817926) it can be a simple way to achieve thread-safety.

But thread-local variables also come with a few caveats. Every thread-local variable is mutable, which makes it hard to discern which component updates shared state and in what order. There's also the risk of memory leaks when using thread-local variables, because unless you call `remove()` on the `ThreadLocal` the data is retained until it is garbage collected (which is only after the thread has finished). And finally, thread-local variables of a parent thread can be inherited by child threads, which results in the child thread having to allocate storage for every thread-local variable previously written in the parent thread.

TODO

#### Typical Use Cases



#### What's Different From Java 19?

[changes since previous preview]

#### More Information

For more information on this feature, see [JEP 429](https://openjdk.org/jeps/429).

### JEP 437: Structured Concurrency (Second Incubator)

[feature summary]

#### What's Different From Java 19?

[changes since previous preview]

#### More Information

For more information on this feature, see [JEP 437](https://openjdk.org/jeps/437).

## From Project Panama

[Project Panama](http://openjdk.java.net/projects/panama/) aims to improve the connection between the JVM and foreign (non-Java) libraries. Java 20 contains two JEPs that originate from Project Panama: 

* Foreign Function & Memory API;
* Vector API.

### JEP 434: Foreign Function & Memory API (Second Preview)

Java programs have always had the option of interacting with code and data outside of the Java runtime.
We could use the [Java Native Interface](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/) (JNI) to invoking foreign functions (outside of the JVM but on the same machine).
And accessing foreign memory (outside of the JVM, so off-heap) was possible using either the [ByteBuffer API](https://docs.oracle.com/en/java/javase/19/docs/api/java.base/java/nio/ByteBuffer.html) or the [sun.misc.Unsafe API](https://github.com/openjdk/jdk/blob/master/src/jdk.unsupported/share/classes/sun/misc/Unsafe.java).

However, these three mechanisms all come with their own drawbacks, which is why a more modern API is now proposed to support foreign functions and foreign memory in a better way.

> Performance-critical libraries like [Tensorflow](https://github.com/tensorflow/tensorflow), [Lucene](https://lucene.apache.org/) or [Netty](https://netty.io/) typicially rely on using foreign memory, because they need more control over the memory they use to prevent the cost and unpredictability that comes with garbage collection.

#### Code Example

In order to demonstrate the new API, [JEP 434](https://openjdk.org/jeps/434) lists a code example that obtains a method handle for a C library function `radixsort` and then uses it to sort four strings which start life in a Java array:

```java
// 1. Find foreign function on the C library path
Linker linker          = Linker.nativeLinker();
SymbolLookup stdlib    = linker.defaultLookup();
MethodHandle radixsort = linker.downcallHandle(stdlib.find("radixsort"), ...);
// 2. Allocate on-heap memory to store four strings
String[] javaStrings = { "mouse", "cat", "dog", "car" };
// 3. Use try-with-resources to manage the lifetime of off-heap memory
try (Arena offHeap = Arena.openConfined()) {
    // 4. Allocate a region of off-heap memory to store four pointers
    MemorySegment pointers = offHeap.allocateArray(ValueLayout.ADDRESS, javaStrings.length);
    // 5. Copy the strings from on-heap to off-heap
    for (int i = 0; i < javaStrings.length; i++) {
        MemorySegment cString = offHeap.allocateUtf8String(javaStrings[i]);
        pointers.setAtIndex(ValueLayout.ADDRESS, i, cString);
    }
    // 6. Sort the off-heap data by calling the foreign function
    radixsort.invoke(pointers, javaStrings.length, MemorySegment.NULL, '\0');
    // 7. Copy the (reordered) strings from off-heap to on-heap
    for (int i = 0; i < javaStrings.length; i++) {
        MemorySegment cString = pointers.getAtIndex(ValueLayout.ADDRESS, i);
        javaStrings[i] = cString.getUtf8String(0);
    }
} // 8. All off-heap memory is deallocated here
assert Arrays.equals(javaStrings, new String[] {"car", "cat", "dog", "mouse"});  // true
```

Let's look at some of the types this code uses in more detail to get a rough idea of their function and purpose within the Foreign Function & Memory API:

`Linker`
: Provides access to foreign functions from Java code, and access to Java code from foreign functions. It allows Java code to link against foreign functions, via *downcall method handles*. It also foreign functions to call Java method handles, via the generation of *upcall stubs*. See the [JavaDoc](https://download.java.net/java/early_access/jdk20/docs/api/java.base/java/lang/foreign/Linker.html) of this type for more information.

`SymbolLookup`
: Retrieves the address of a symbol in one or more libraries. See the [JavaDoc](https://download.java.net/java/early_access/jdk20/docs/api/java.base/java/lang/foreign/SymbolLookup.html) of this type for more information.

`Arena`
: Controls the lifecycle of memory segments. An arena has a scope, called the arena scope. When the arena is closed, the arena scope is no longer alive. As a result, all the segments associated with the arena scope are invalidated, safely and atomically, their backing memory regions are deallocated (where applicable) and can no longer be accessed after the arena is closed. See the [JavaDoc](https://download.java.net/java/early_access/jdk20/docs/api/java.base/java/lang/foreign/Arena.html) of this type for more information.

`MemorySegment`
: Provides access to a contiguous region of memory. There are two kinds of memory segments: *heap segments* (inside the Java memory heap) and *native segments* (outside of the Java memory heap). See the [JavaDoc](https://download.java.net/java/early_access/jdk20/docs/api/java.base/java/lang/foreign/MemorySegment.html) of this type for more information. 

`ValueLayout`
: Models values of basic data types, such as *integral* values, *floating-point* values and *address* values. On top of that, it defines useful value layout constants for Java primitive types and addresses. See the [JavaDoc](https://download.java.net/java/early_access/jdk20/docs/api/java.base/java/lang/foreign/ValueLayout.html) of this type for more information.

#### What's Different From Java 19?

In Java 19, this feature was in its first preview status (in the form of [JEP 424](https://openjdk.org/jeps/424)), so the language feature was complete and developer feedback was gathered. Based on this feedback the following changes happened to Java 20:

* The `MemorySegment` and `MemoryAddress` abstractions were unified (memory addresses are now modeled by zero-length memory segments);
* The sealed `MemoryLayout` hierarchy was enhanced to facilitate usage with pattern matching in switch expressions and statements (see [JEP 433](#jep-433-pattern-matching-for-switch-fourth-preview));
* `MemorySession` has been split into `Arena` and `SegmentScope` to facilitate sharing segments across maintenance boundaries.

#### More Information

For more information on this feature, see [JEP 434](https://openjdk.org/jeps/434).

### JEP 438: Vector API (Fifth Incubator)

The Vector API will make it possible to express vector computations that reliably compile at runtime to optimal vector instructions. 
This means that these computations will significantly outperform equivalent scalar computations on the supported CPU architectures (x64 and AArch64).

#### Vector Computations? Help Me Out Here!

A *vector computation* is a mathematical operation on one or more one-dimensional matrices of an arbitrary length. Think of a vector as an array with a dynamic length. Furthermore, the elements in the vector can be accessed in constant time via indices, just like with an array. 

In the past, Java programmers could only program such computations at the assembly-code level. But now that modern CPUs support advanced [SIMD](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data) features (Single Instruction, Multiple Data), it becomes more important to take advantage of the performance gain that SIMD instructions and multiple lanes operating in parallel can bring. The Vector API brings that possibility closer to the Java programmer.

#### Typical Use Cases

The Vector API provides a way to write complex vector algorithms in Java that perform extremely well, such as vectorized `hashCode` implementations or specialized array comparisons. Numerous domains can benefit from this, including machine learning, linear algebra, encryption, text processing, finance, and code within the JDK itself.

#### What's Different From Java 19?

Aside from a small set of bug fixes and performance enhancements, the alignment of this feature with [Project Valhalla](https://openjdk.org/projects/valhalla/) is the biggest difference with Java 19. And it's one that makes a lot of sense, as both the Vector API and Project Valhalla focus on performance improvements. 

> Recall that Project Valhalla's aim is to augment the Java object model with value objects and user-defined primitives, combining the abstractions of object-oriented programming with the performance characteristics of simple primitives. 

Once the features of Project Valhalla are available, the Vector API will be adapted to make use of value objects and by that time it will be promoted to a preview feature.

#### More Information

For more information on this feature, see [JEP 438](https://openjdk.org/jeps/438).

## Wrap-up

...
