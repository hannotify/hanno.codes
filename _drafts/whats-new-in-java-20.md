---
layout: post
title: It's Java 20 Release Day! Here's What's New
date: 21-03-2023
header:
  teaser: /assets/images/blog/package-just-for-you.jpg
tags: 
- java
---

This will be a good day, because it's Java 20 release day! It's been six months since Java 19 was released, and so it's time for another fresh wave of Java features. This post will take you on a tour through all JEPs that are associated with this release and give you a brief introduction of each one of them. Where applicable the differences with Java 19 are highlighted and a few typical use cases are provided, so that you'll be more than ready to use these features after you've finished reading!

![Package, just for you](/assets/images/blog/package-just-for-you.jpg)
> Image from <a href="https://pxhere.com/nl/photo/535806">PxHere</a>

## From Project Amber

Java 20 contains two features that originated from [Project Amber](https://openjdk.org/projects/amber/):

* Pattern Matching for switch;
* Record Patterns.

> The goal of Project Amber is to explore and incubate smaller, productivity-oriented Java language features.

### JEP 433: Pattern Matching for switch (Fourth Preview)

The feature 'Pattern Matching for switch' that was first introduced in Java 17 has reached its fourth preview stage, now that Java 20 has been released. The feature has always gathered a lot of feedback in the past, and on top of that it needs alignment with the related Record Patterns preview feature. In the end there are lots of reasons to keep it in preview just a little longer.

Since Java 16 we are able to avoid casting after `instanceof` checks by using 'Pattern Matching for instanceof'. Let's see how that works in a code example.

> Code examples that illustrate this JEP were taken from my conference talk ["Pattern Matching: Small Enhancement or Major Feature?"](https://hanno.codes/talks/#pattern-matching-small-enhancement-or-major-feature).

```java
static String apply(Effect effect) {
    String formatted = "";
    if (effect instanceof Delay de) {
        formatted = String.format("Delay active of %d ms.", de.getTimeInMs());
    } else if (effect instanceof Reverb re) {
        formatted = String.format("Reverb active of type %s and roomSize %d.", re.getName(), re.getRoomSize());
    } else if (effect instanceof Overdrive ov) {
        formatted = String.format("Overdrive active with gain %d.", ov.getGain());
    } else if (effect instanceof Tremolo tr) {
        formatted = String.format("Tremolo active with depth %d and rate %d.", tr.getDepth(), tr.getRate());
    } else if (effect instanceof Tuner tu) {
        formatted = String.format("Tuner active with pitch %d. Muting all signal!", tu.getPitchInHz());
    } else {
        formatted = String.format("Unknown effect active: %s.", effect);
    }
    return formatted;
}
```

This code is still riddled with ceremony, though. On top of that it leaves room for subtle bugs — what if you added an else-if branch that didn't assign anything to `formatted`? So in the spirit of this JEP (and its predecessors), let's see what pattern matching in a switch statement (or even better: in a switch *expression*) would look like:

```java
static String apply(Effect effect) {
    return switch(effect) {
        case Delay de      -> String.format("Delay active of %d ms.", de.getTimeInMs());
        case Reverb re     -> String.format("Reverb active of type %s and roomSize %d.", re.getName(), re.getRoomSize());
        case Overdrive ov  -> String.format("Overdrive active with gain %d.", ov.getGain());
        case Tremolo tr    -> String.format("Tremolo active with depth %d and rate %d.", tr.getDepth(), tr.getRate());
        case Tuner tu      -> String.format("Tuner active with pitch %d. Muting all signal!", tu.getPitchInHz());
        case null, default -> String.format("Unknown or empty effect active: %s.", effect);
    };
}
```

Pattern matching for switch made our code far more elegant here. We're even able to address possible `null`s by defining a specific case for it or combining it with the default (which is what we've done here).

Checking an additional condition on top of the pattern match is easily done with a *guard* (the part after the `when` keyword in the code below):

```java
static String apply(Effect effect, Guitar guitar) {
    return switch(effect) {
        case Delay de      -> String.format("Delay active of %d ms.", de.getTimeInMs());
        case Reverb re     -> String.format("Reverb active of type %s and roomSize %d.", re.getName(), re.getRoomSize());
        case Overdrive ov  -> String.format("Overdrive active with gain %d.", ov.getGain());
        case Tremolo tr    -> String.format("Tremolo active with depth %d and rate %d.", tr.getDepth(), tr.getRate());
        case Tuner tu when !guitar.isInTune() -> String.format("Tuner active with pitch %d. Muting all signal!", tu.getPitchInHz());
        case Tuner tu      -> "Guitar is already in tune.";
        case null, default -> String.format("Unknown or empty effect active: %s.", effect);
    };
}
```

Here, the guard makes sure that intricate boolean logic can still be expressed concisely. Having to nest `if` statements to test this logic within a case branch would not only be more verbose, but also potentially introduce subtle bugs that we set out to avoid in the first place.

#### What's Different From Java 19?

A few changes were made to this feature compared to Java 19:

* When a pattern match fails abruptly, a `MatchException` is now thrown.
* Inference of type arguments for [record patterns](#jep-432-record-patterns-second-preview) is now supported in switch expressions and statements. This means that you can now use `var` in the patterns you want to match.

#### More Information

For more information on this feature, see [JEP 433](https://openjdk.org/jeps/433).

### JEP 432: Record Patterns (Second Preview)

[feature summary]

#### What's Different From Java 19?

[changes since previous preview]

#### More Information

For more information on this feature, see [JEP 432](https://openjdk.org/jeps/432).

## From Project Loom

Java 20 contains three features that originated from [Project Loom](http://openjdk.java.net/projects/loom/):

* Virtual Threads;
* Scoped Values;
* Structured Concurrency.

> Project Loom strives to simplify maintaining concurrent applications in Java by introducing *virtual threads* and an API for *structured concurrency*, among other things.

### JEP 436: Virtual Threads (Second Preview)

Threads have been a part of Java since the very beginning, and since the start of Project Loom we've gradually started calling these 'classic' threads 'platform threads' instead. A platform thread runs Java code on an underlying OS thread and captures the OS thread for the code's entire lifetime. The number of platform threads is therefore limited to the number of available OS threads.

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

When your code uses the `ExecutorService` interface already, switching to virtual threads can take even less effort:

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

Note that the `ExecutorService` interface was  adjusted in Java 19 to extend `AutoCloseable`, so it can now be used in a try-with-resources construct.

#### What's Different From Java 19?

The feature is in the 'second preview' stage, to allow for more feedback. On top of that, a few API changes were made permanent and as a result are not proposed for preview any longer. They've been made permanent because they involve functionality that is useful in general and is not specific to virtual threads, including:

* new methods in `Thread` class:
  * `join(Duration)`; 
  * `sleep(Duration)`; 
  * `threadId()`. 
* new methods in `Future` (to examine task state and result)
* the change to make `ExecutorService` extend `AutoCloseable`, so that it can be used in a try-with-resources block.

On top of that, [the degradations to `ThreadGroup`](https://openjdk.org/jeps/425#java-lang-ThreadGroup) were also made permanent.

#### More Information

For more information on this feature, see [JEP 436](https://openjdk.org/jeps/436).

### JEP 429: Scoped Values (Incubator)

*Scoped values* enable the sharing of immutable data within and across threads.
They are preferred to thread-local variables, especially when using large numbers of virtual threads.

#### ThreadLocal

Since Java 1.2 we can make use of `ThreadLocal` variables, which confine a certain value to the thread that created it.
In [some cases](https://stackoverflow.com/a/817926) it can be a simple way to achieve thread-safety.

But thread-local variables also come with a few caveats. Every thread-local variable is mutable, which makes it hard to discern which component updates shared state and in what order. There's also the risk of memory leaks, because unless you call `remove()` on the `ThreadLocal` the data is retained until it is garbage collected (which is only after the thread has terminated). And finally, thread-local variables of a parent thread can be inherited by child threads, which results in the child thread having to allocate storage for every thread-local variable previously written in the parent thread.

These drawbacks become more apparent now that virtual threads have been introduced, because millions of them could be active at the same time - each with their own thread-local variables - which would result in a very significant memory footprint.

#### Scoped Values

Like a thread-local variable, a scoped value has multiple incarnations, one per thread. Unlike a thread-local variable, a scoped value is written once and is then immutable, and is available only for a bounded period during execution of the thread.

The JEP illustrates the use of scoped values with the pseudo code example below:

```java
final static ScopedValue<...> V = new ScopedValue<>();

// In some method
ScopedValue.where(V, <value>)
           .run(() -> { ... V.get() ... call methods ... });

// In a method called directly or indirectly from the lambda expression
... V.get() ...
```

We see that `ScopedValue.where(...)` is called, presenting a scoped value and the object to which it is to be bound. The call to `run(...)` binds the scoped value, providing an incarnation that is specific to the current thread, and then executes the lambda expression passed as argument. During the lifetime of the `run(...)` call, the lambda expression, or any method called directly or indirectly from that expression, can read the scoped value via the value’s `get()` method. After the `run(...)` method finishes, the binding is destroyed.

#### Typical Use Cases

Scoped values will be useful in all places where currently thread-local variables are used for the purpose of one-way transmission of unchanging data. 

#### What's Different From Java 19?

Java 19 didn't contain anything related to scoped values yet, so Java 20 is the first time we get to experiment with scoped values. Note that the JEP is in the [incubator](https://openjdk.org/jeps/11) stage, so you'll need to add `--enable-preview --add-modules jdk.incubator.concurrent` to the command-line to be able to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 429](https://openjdk.org/jeps/429).

### JEP 437: Structured Concurrency (Second Incubator)

Java's current implementation of concurrency is *unstructured*, which makes error handling and cancellation with multiple tasks a challenge. When multiple tasks are started up asynchronously, we currently aren't able to cancel the remaining tasks once the first task returns an error.

Let's illustrate this point with the code example from the JEP: 

```java
Response handle() throws ExecutionException, InterruptedException {
    Future<String>  user = executor.submit(() -> findUser());
    Future<Integer> order = executor.submit(() -> fetchOrder());
    String theUser = user.get();   // Join findUser
    int theOrder = order.get();  // Join fetchOrder
    return new Response(theUser, theOrder);
}
```

When the `user.get()` call results in an error, there is no way for us to cancel the second task to prevent getting a result that won't be needed anyway. 
Though when we would rewrite this code to use just a single thread, the situation becomes a lot simpler:

```java
Response handle() throws IOException {
    String theUser = findUser();
    int theOrder = fetchOrder();
    return new Response(theUser, theOrder);
}
```

Because here we would be able to prevent the second call once the first one has failed.

In general, multithreaded programming in Java would be easier, more reliable, and more observable if the parent-child relationships between tasks and their subtasks were expressed syntactically — just as for single-threaded code. The syntactic structure would delineate the lifetimes of subtasks and enable a runtime representation of the inter-thread hierarchy, enabling error propagation and cancellation as well as meaningful observation of the concurrent program.

Enter *structured concurrency*. We've now rewritten the code example to make use of the new `StructuredTaskScope` API:

```java
Response handle() throws ExecutionException, InterruptedException {
  try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
      Future<String>  user  = scope.fork(() -> findUser());
      Future<Integer> order = scope.fork(() -> fetchOrder());

      scope.join();           // Join both forks
      scope.throwIfFailed();  // ... and propagate errors

      // Here, both forks have succeeded, so compose their results
      return new Response(user.resultNow(), order.resultNow());
  }
}
```

In structured concurrency, subtasks work on behalf of a task. The task awaits the subtasks' results and monitors them for failures. The `StructuredTaskScope` class allows developers to structure a task as a family of concurrent subtasks, and to coordinate them as a unit. Subtasks are executed in their own threads by forking them individually and then joining them as a unit and, possibly, cancelling them as a unit. The subtasks' successful results or exceptions are aggregated and handled by the parent task. 

In contrast to the original example, understanding the lifetimes of the threads involved here is easy. Under all conditions their lifetimes are confined to a lexical scope, namely the body of the try-with-resources statement. Furthermore, the use of `StructuredTaskScope` ensures a number of valuable properties:

* *Error handling with short-circuiting*. If either the findUser() or fetchOrder() subtasks fail, the other is cancelled if it has not yet completed. This is managed by the cancellation policy implemented by `ShutdownOnFailure`; other policies like `ShutdownOnSuccess` are also available.

* *Cancellation propagation*. If the thread running handle() is interrupted before or during the call to join(), both forks are cancelled automatically when the thread exits the scope.

* *Clarity* — The above code has a clear structure: Set up the subtasks, wait for them to either complete or be cancelled, and then decide whether to succeed (and process the results of the child tasks, which are already finished) or fail (and the subtasks are already finished, so there is nothing more to clean up).

By the way: it is by no means coincidental that structured concurrency is coming to Java at the same time as virtual threads. Modern Java programs will likely use an abundance of threads, and they need to be correctly and robustly coordinated. Structured concurrency can provide exactly that, while also enabling observability tools to display threads as they are understood by the developer.

#### What's Different From Java 19?

The situation is roughly the same to how it was in Java 19 (see [JEP 428](https://openjdk.org/jeps/428)).
The only change is an update to `StructuredTaskScope` to make it support the inheritance of [scoped values](#jep-429-scoped-values-incubator) by threads created in a task scope. This streamlines the sharing of immutable data across threads. Note that the JEP is in the [incubator](https://openjdk.org/jeps/11) stage, so you'll need to add `--enable-preview --add-modules jdk.incubator.concurrent` to the command-line to be able to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 437](https://openjdk.org/jeps/437).

## From Project Panama

Java 20 contains two features that originated from [Project Panama](http://openjdk.java.net/projects/panama/):

* Foreign Function & Memory API;
* Vector API.

> Project Panama aims to improve the connection between the JVM and foreign (non-Java) libraries.

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

...TODO
