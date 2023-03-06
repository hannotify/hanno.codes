---
layout: post
title: It's Java 20 Release Day! Here's What's New
date: 21-03-2023
header:
  teaser:
tags: 
- java
---

It's the week of Java 20's release! ...

## On Feature Statuses

But let's get into these feature statuses for a bit.
A feature status can be *preview*, *incubating* or *experimental*, but the differences between them are quite subtle.

### Preview

* Complete or almost complete language features ready to try out.
* Serve to collect feedback about the features without committing to maintaining its backward compatibility. This means everyone is encouraged to try them out, but, at the same time, discouraged from using them in production.
* Relatively short “grace-period” so you can expect them to mature fast.
* Requires the `--enable-preview` compiler flag.

### Incubating

* Like 'experimental', but distributed in a form of separate modules with names prefixed with `jdk.incubator`.
* In order to access them, they need to be added to the module path explicitly.

### Experimental

* Represent early versions of (mostly) VM-level features, which can be risky, incomplete, or even unstable. 
* Need to be enabled using dedicated flags.
* For the purpose of comparison, if an experimental feature is considered 25% “done”, then a preview feature should be at least 95% “done”.

<https://openjdk.org/jeps/11>

## From Project Amber

[short Amber intro]

## JEP 432: Record Patterns (Second Preview)

[feature summary]
[changes since previous preview]

## JEP 433: Pattern Matching for switch (Fourth Preview)

The feature 'Pattern Matching for switch' that was first introduced in Java 16 has reached its fourth preview status now that Java 20 has been released.
...

[feature summary]
[changes since previous preview]

## From Project Loom

[short Loom intro]

### JEP 429: Scoped Values (Incubator)

[feature summary]

### JEP 436: Virtual Threads (Second Preview)

[feature summary]

#### What changed since Java 19?

[changes since previous preview]

### JEP 437: Structured Concurrency (Second Incubator)

[feature summary]

#### What changed since Java 19?

[changes since previous preview]

## From Project Panama

[short Panama intro]

### JEP 434: Foreign Function & Memory API (Second Preview)

Java programs have always had the option of interacting with code and data outside of the Java runtime.
Invoking foreign functions (outside of the JVM but on the same machine) could be done through the Java Native Interface (JNI).
And accessing foreign memory (outside of the JVM, so off-heap) was possible using either the [ByteBuffer API](https://docs.oracle.com/en/java/javase/19/docs/api/java.base/java/nio/ByteBuffer.html) or the [sun.misc.Unsafe API](https://github.com/openjdk/jdk/blob/master/src/jdk.unsupported/share/classes/sun/misc/Unsafe.java).
Performance-critical libraries like [Tensorflow](https://github.com/tensorflow/tensorflow), [Lucene](https://lucene.apache.org/) or [Netty](https://netty.io/) rely on using foreign memory, because they need more control over the memory they use to prevent the cost and unpredictability that comes with garbage collection.
However, these three mechanisms all come with their own drawbacks, which is why a more modern API is now proposed to support foreign functions and foreign memory in a better way.

#### Code example

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

These are the 




#### What's Different From Java 19?

In Java 19, this feature was in its first preview status (in the form of [JEP 424](https://openjdk.org/jeps/424)), so the language feature was complete and developer feedback was gathered.
This resulted in the following changes:

* The `MemorySegment` and `MemoryAddress` abstractions are unified (memory addresses are now modeled by zero-length memory segments);
* The sealed `MemoryLayout` hierarchy is enhanced to facilitate usage with pattern matching in switch expressions and statements (see [JEP 433](#jep-433-pattern-matching-for-switch-fourth-preview));
* `MemorySession` has been split into `Arena` and `SegmentScope` to facilitate sharing segments across maintenance boundaries.

### JEP 438: Vector API (Fifth Incubator)

TODO [feature summary]

#### What changed since Java 19?

[changes since previous preview]

## Wrap-up

...
