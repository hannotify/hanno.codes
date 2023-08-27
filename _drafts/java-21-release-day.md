---
layout: post
title: Java 21 is Available Today! Here's What's Inside
date: 19-09-2023 04:30:00 +0200
header:
  teaser: /assets/images/blog/ready-set-go.jpg
tags: 
- java
---

Today's the first day of Java 21's availability! It's been six months since Java 20 was released, and so it's time for another fresh wave of Java features. This post takes you on a tour of the JEPs that are associated with this release and it gives you a brief introduction to each one of them. Where applicable the differences with Java 20 are highlighted and a few typical use cases are provided, so that you'll be more than ready to use these features after you've finished reading!

![Ready, set, go!](/assets/images/blog/ready-set-go.jpg)
> Image from <a href="https://pxhere.com/en/photo/47097">PxHere</a>

## From Project Amber

Java 21 contains five features that originated from [Project Amber](https://openjdk.org/projects/amber/):

* String Templates;
* Pattern Matching for switch;
* Record Patterns;
* Unnamed Patterns and Variables;
* Unnamed Classes and Instance Main Methods.

> The goal of Project Amber is to explore and incubate smaller, productivity-oriented Java language features.

### JEP 430: String Templates (Preview)

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 430](https://openjdk.org/jeps/430).

### JEP 441: Pattern Matching for switch

The feature 'Pattern Matching for switch' that was first introduced in Java 17 has reached completion status, now that Java 21 has been released.

Since Java 16 we have been able to avoid casting after `instanceof` checks by using 'Pattern Matching for instanceof'. Let's see how that works in a code example.

> All code examples that illustrate pattern matching features were taken from my conference talk ["Pattern Matching: Small Enhancement or Major Feature?"](https://hanno.codes/talks/#pattern-matching-small-enhancement-or-major-feature).

```java
static String apply(Effect effect) {
    String formatted = "";
    if (effect instanceof Delay de) {
        formatted = String.format("Delay active of %d ms.", de.timeInMs());
    } else if (effect instanceof Reverb re) {
        formatted = String.format("Reverb active of type %s and roomSize %d.", re.name(), re.roomSize());
    } else if (effect instanceof Overdrive ov) {
        formatted = String.format("Overdrive active with gain %d.", ov.gain());
    } else if (effect instanceof Tremolo tr) {
        formatted = String.format("Tremolo active with depth %d and rate %d.", tr.depth(), tr.rate());
    } else if (effect instanceof Tuner tu) {
        formatted = String.format("Tuner active with pitch %d. Muting all signal!", tu.pitchInHz());
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
        case Delay de      -> String.format("Delay active of %d ms.", de.timeInMs());
        case Reverb re     -> String.format("Reverb active of type %s and roomSize %d.", re.name(), re.roomSize());
        case Overdrive ov  -> String.format("Overdrive active with gain %d.", ov.gain());
        case Tremolo tr    -> String.format("Tremolo active with depth %d and rate %d.", tr.depth(), tr.rate());
        case Tuner tu      -> String.format("Tuner active with pitch %d. Muting all signal!", tu.pitchInHz());
        case null, default -> String.format("Unknown or empty effect active: %s.", effect);
    };
}
```

Pattern matching for switch made our code far more elegant here. We're even able to address possible `null`s by defining a specific case for them or combining it with the default case (which is what we've done here).

Checking an additional condition on top of the pattern match is easily done with a *guard* (the part after the `when` keyword in the code below):

```java
static String apply(Effect effect, Guitar guitar) {
    return switch(effect) {
        case Delay de      -> String.format("Delay active of %d ms.", de.timeInMs());
        case Reverb re     -> String.format("Reverb active of type %s and roomSize %d.", re.name(), re.roomSize());
        case Overdrive ov  -> String.format("Overdrive active with gain %d.", ov.gain());
        case Tremolo tr    -> String.format("Tremolo active with depth %d and rate %d.", tr.depth(), tr.rate());
        case Tuner tu when !guitar.isInTune() -> String.format("Tuner active with pitch %d. Muting all signal!", tu.pitchInHz());
        case Tuner tu      -> "Guitar is already in tune.";
        case null, default -> String.format("Unknown or empty effect active: %s.", effect);
    };
}
```

Here, the guard makes sure that intricate boolean logic can still be expressed in a concise way. Having to nest `if` statements to test this logic within a case branch would not only be more verbose, but also potentially introduce subtle bugs that we set out to avoid in the first place.

#### What's Different From Java 20?

There have been two major changes from the previous JEP:

* An earlier version of the 'Pattern Matching for switch' feature came with [parenthesized patterns](https://docs.oracle.com/en/java/javase/17/language/pattern-matching.html#GUID-A59EF0C7-4CB7-4555-986D-0FD804555C25), which helped resolve parsing ambiguities back when guards were still expressed with the `&&` operator. Now that the `when` keyword has replaced the `&&` operator for guards, the value for parenthesized patterns has decreased significantly. So the choice was made to remove them in Java 21.
* Switch expressions and statements now allow qualified enum constants as case constants. This makes it easier to perform an exhaustive `switch` over an interface type when both a class and an enum implementation of that interface exists. The JEP description has [a good example of this mechanism](https://openjdk.org/jeps/441#Switches-and-enum-constants), should you wish to learn more about it.    

#### More Information

For more information on this feature, see [JEP 441](https://openjdk.org/jeps/441).

### JEP 440: Record Patterns

Pattern matching is a feature in Java that is being rolled out gradually over multiple Java versions. Being able to deconstruct an object using patterns was always one of the ultimate goals of the feature arc. With the introduction of *record patterns*, deconstructing records is now possible, along with nesting record and type patterns to enable a powerful, declarative, and composable form of data navigation and processing.

[Records](https://openjdk.org/jeps/395) are transparent carriers for data. Code that receives an instance of a record will typically extract the data, known as the components. This was also the case in our 'Pattern Matching for switch' code example, if we assume that all implementations of the `Effect` interface were in fact records there. In that piece of code it is clear that the pattern variables only serve to access the record fields. Using record patterns we can avoid having to create pattern variables altogether:

```java
static String apply(Effect effect) {
    return switch(effect) {
        case Delay(int timeInMs) -> String.format("Delay active of %d ms.", timeInMs);
        case Reverb(String name, int roomSize) -> String.format("Reverb active of type %s and roomSize %d.", name, roomSize);
        case Overdrive(int gain) -> String.format("Overdrive active with gain %d.", gain);
        case Tremolo(int depth, int rate) -> String.format("Tremolo active with depth %d and rate %d.", depth, rate);
        case Tuner(int pitchInHz) -> String.format("Tuner active with pitch %d. Muting all signal!", pitchInHz);
        case null, default -> String.format("Unknown or empty effect active: %s.", effect);
    };
}
```

`Delay(int timeInMs)` is a record pattern here, deconstructing the `Delay` instance into its components. And this mechanism can become even more powerful when we apply it to a more complicated object graph by using *nested* record patterns:

```java
record Tuner(int pitchInHz, Note note) implements Effect {}
record Note(String note) {}

class TunerApplier {
    static String apply(Effect effect, Guitar guitar) {
        return switch(effect) {
            case Tuner(int pitch, Note(String note)) -> String.format("Tuner active with pitch %d on note %s", pitch, note);
        };
    }
}
```
#### Inference of type arguments

Nested record patterns also benefit from *inference of type arguments*. For example:

```java
class TunerApplier {
    static String apply(Effect effect, Guitar guitar) {
        return switch(effect) {
            case Tuner(var pitch, Note(var note)) -> String.format("Tuner active with pitch %d on note %s", pitch, note);
        };
    }
}
```

Here the type arguments for the nested pattern `Tuner(var pitch, Note(var note))` are inferred. This only works with nested patterns for now; type patterns do not yet support implicit inference of type arguments. So the type pattern `Tuner tu` is always treated as a raw type pattern.

#### What's Different From Java 20?

Apart from some minor editorial changes, the main change since the second preview is to remove support for record patterns appearing in the header of an enhanced for statement. This feature may be re-proposed in a future JEP.

TODO: why?

#### More Information

For more information on this feature, see [JEP 440](https://openjdk.org/jeps/440).

### JEP 443: Unnamed Patterns and Variables (Preview)

TODO

#### What's Different From Java 20?

Java 20 didn't contain anything related to unnamed patterns and variables yet, so Java 21 is the first time we get to experiment with them. Note that the JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to be able to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 443](https://openjdk.org/jeps/443).

### JEP 445: Unnamed Classes and Instance Main Methods (Preview)

TODO

#### What's Different From Java 20?

Java 20 didn't contain anything related to unnamed classes and instance methods yet, so Java 21 is the first time we get to experiment with them. Note that the JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to be able to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 445](https://openjdk.org/jeps/445).

## From Project Loom

Java 21 contains three features that originated from [Project Loom](http://openjdk.java.net/projects/loom/):

* Virtual Threads;
* Scoped Values;
* Structured Concurrency.

> Project Loom strives to simplify maintaining concurrent applications in Java by introducing *virtual threads* and an API for *structured concurrency*, among other things.

### JEP 444: Virtual Threads

Threads have been a part of Java since the very beginning, and since the start of Project Loom we gradually started calling them 'platform threads' instead. A platform thread runs Java code on an underlying OS thread and captures the OS thread for the code's entire lifetime. The number of platform threads is therefore limited to the number of available OS threads.

Modern applications, however, might need many more threads than that; when dealing with tens of thousands of requests at the same time, for example. This is where *virtual threads* come in. A virtual thread is an instance of `java.lang.Thread` that also runs Java code on an underlying OS thread, but does not capture the OS thread for the code's entire lifetime. This means that many virtual threads can run their Java code on the same OS thread, effectively sharing it. The number of virtual threads can thus be much larger than the number of available OS threads.

Aside from being plentiful, virtual threads are also cheap to create and dispose of. This means that a web framework, for example, can dedicate a new virtual thread to the task of handling a request and still be able to process thousands or even millions of requests at once.

#### Typical Use Cases

Using virtual threads does not require learning new concepts, though it may require unlearning habits developed to cope with today's high cost of threads. Virtual threads will not only help application developers; they will also help framework designers provide easy-to-use APIs that are compatible with the platform's design without compromising on scalability.

#### Creating Virtual Threads

Just like a platform thread, a virtual thread is an instance of `java.lang.Thread`. So you can use a virtual thread in exactly the same way as a platform thread.

Creating a virtual thread is a bit different from creating a platform thread, but just as easy:

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

#### Thread Dump Mechanism

A new kind of thread dump in `jcmd` was introduced to present virtual threads alongside platform threads, all grouped in a meaningful way. This new way of presenting threads was needed, because of the way the JDK's traditional thread dump (obtained through `jstack` or `jcmd`) presents a flat list of threads. The old format worked fine with hundreds of platform threads, but it is unsuitable for thousands or millions of virtual threads. An additional benefit of the new thread dump is the ability to show richer relationships amoung threads when programs use [structured concurrency](#jep-453-structured-concurrency-preview).

#### What's Different From Java 20?

Based on developer feedback the following changes were made to virtual threads compared to Java 20:

* Virtual threads now always support thread-local variables. Guaranteed support for thread-local variables ensures that many more existing libraries can be used unchanged with virtual threads, and this helps with the migration of task-oriented code to use virtual threads.
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

## HotSpot

Java 21 introduces three changes to [HotSpot](https://openjdk.org/groups/hotspot/):

* Generational ZGC;
* Deprecate the Windows 32-bit x86 Port for Removal;
* Prepare to Disallow the Dynamic Loading of Agents.

> The HotSpot JVM is the runtime engine that is developed by Oracle. It translates Java bytecode into machine code for the host operating system's processor architecture.

### JEP 439: Generational ZGC

The Z Garbage Collector (ZGC) is a scalable, low-latency garbage collector. It has been [available for production use since Java 15](https://openjdk.org/jeps/377) and has been designed to keep pause times consistent and short, even for very large heaps. It uses techniques like region-based memory management and compaction to achieve this.

Java 21 introduces an extension to ZGC that maintains separate *generations* for young and old objects, allowing ZGC to collect young objects (which tend to die young) more frequently. This will result in a significant performance gain for applications running with generational ZGC, without sacrificing any of the valuable properties that the Z garbage collector is already known for:

* Pause times do not exceed 1 millisecond;
* Heap sizes from a few hundred megabytes up to many terabytes are supported;
* Minimal manual configuration is needed.

The reason for handling young and old objects separately stems from the [weak generational hypothesis](#GUID-71D796B3-CBAB-4D80-B5C3-2620E45F6E5D), stating that young objects tend to die young, while old objects tend to stick around. This means that collecting young objects requires fewer resources and yields more memory, while collecting old objects requires more resources and yields less memory. This is the reason we can improve the performance of applications that use ZGC by collecting young objects more frequently.

#### What's Different From Java 20?

The Z garbage collector in Java 20 was only able to behave in a non-generational way. Running it required the following command-line configuration:

`$ java -XX:+UseZGC ...`

To run your workload with the new Generational ZGC in Java 21, use the following configuration:

`$ java -XX:+UseZGC -XX:+ZGenerational ...`

As you can see, Generational ZGC has been introduced *alongside* non-generational ZGC. In a future release we can expect Generational ZGC to become the default configuration for the Z garbage collector, while an even later release will probably remove non-generational ZGC altogether.

#### More Information

For more information on this feature, see [JEP 439](https://openjdk.org/jeps/439).

### JEP 449: Deprecate the Windows 32-bit x86 Port for Removal

Supporting multiple platforms has been the focus of the Java ecosystem since the beginning. 
But older platforms cannot be supported indefinitely, and that is one of the reasons why the Windows 32-bit x86 port is now scheduled for removal.

There are a few reasons for this:

* Windows 10, the last Windows operating system to support 32-bit operation, [will reach end-of-life in October 2025](https://learn.microsoft.com/lifecycle/products/windows-10-home-and-pro);
* The implementation of virtual threads on Windows 32-bit x86 is rudimentary to say the least: it effectively uses a single platform thread for each virtual thread, effectively rendering the feature useless on this platform;
* It will allow the OpenJDK contributors to accelerate the development of new features and enhancements.

#### What's Different From Java 20?

Configuring a Windows x86-32 build will now fail on JDK 21. 
This error can be suppressed by using the new build configuration option `--enable-deprecated-ports=yes`.
Note that this currently still means Windows x86-32 users can still use JDK 21; however a future release will actually remove the support, and by that time the affected users are expected to migrate to Windows x64 and a 64-bit JVM. 

#### More Information

For more information about this deprecation, see [JEP 449](https://openjdk.org/jeps/449).

### JEP 451: Prepare to Disallow the Dynamic Loading of Agents

An *agent* is a component that can alter the code of a Java application while it is running. Introduced in JDK 5, agents provide a way for tools (such as profilers) to instrument classes, with the aim of altering the code in a class so that it emits events to be consumed by a tool outside the application. *Dynamically loaded agents* grant serviceability tools the capability to modify a *running* application. However, this capability is available to both tools and libraries, and it can just as easily be used for modifications with bad intentions. To assure integrity, stronger measures are needed to prevent the misuse by libraries of dynamically loaded agents. JEP 451 therefore proposes to require the dynamic loading of agents to be approved by the application owner. This means that in Java 21, the application owner will have to explicitly allow the dynamic loading of agents via a command-line option.

#### What's Different From Java 20?

Java 21 is the first version of Java that issues warnings when agents are loaded dynamically into a running JVM. A future release of the JDK will, by default, disallow the mechanism. Agents that are loaded at startup will still be allowed though, so serviceability tools that use that mechanism won't be affected now or in the future. Maintainers of libraries which rely on dynamically agent loading will have to update their documentation to ask application owners for permission to load the agent at startup.

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
