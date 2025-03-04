---
layout: post
title: Java 24 Rolls Out Today! Find Out Why It's Aptly Named
date: 18-03-2025 04:30:00 +0200
header:
  teaser: /assets/images/blog/basketball-24.jpg
excerpt: Java 24 rolls out today, and it brings a diverse set of features. For example, compact object headers bring better performance, and various security features have been added. Or how about the eagerly-awaited solution to virtual thread pinning? This post has all the info!
tags: 
- java
---

Java 24 rolls out today! It's been six months since Java 23 was released, so it's time for another helping of new features. And this particular release of Java is aptly named, because it contains exactly 24 JEPs. Coincidence? I think not. 🙂

Java 24 brings a diverse set of features, delivering performance improvements like compact object headers, garbage collection optimizations and the first JEP to come out of Project Leyden. On top of that, various security features related to the quantum computing field were added, and a solution to virtual thread pinning is now available!

Apart from these, a few new features from older releases have been repreviewed. 

> Short descriptions of the repreviewed features are provided to prevent this article from becoming a bit too lengthy. Each repreviewed feature has a link to a longer description of the feature should you wish to learn more.

![Basketball players hugging during game - one of them wears a jersey with number '24' at the back](/assets/images/blog/basketball-24.jpg)
> Photo by Royy Nguyen, from <a href="https://www.pexels.com/photo/basketball-players-hugging-during-game-in-gym-5303477/">Pexels</a>

## JEP Overview

To start off, let's look at an overview of the JEPs that ship with Java 24. This table contains the preview status for all JEP's, to which project they belong, what kind of features they add and the things that have changed since Java 23.

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

Let's start with the JEP's that add brand-new features to Java 24.

### HotSpot

Java 24 introduces five new features in [HotSpot](https://openjdk.org/groups/hotspot/):

* Generational Shenandoah (Experimental)
* Compact Object Headers (Experimental)
* Late Barrier Expansion for G1
* Ahead-of-Time Class Loading & Linking
* Synchronize Virtual Threads Without Pinning

> The HotSpot JVM is the runtime engine that is developed by Oracle. It translates Java bytecode into machine code for the host operating system's processor architecture.

#### JEP 404: Generational Shenandoah (Experimental)

The Shenandoah garbage collector is an ultra-low pause time garbage collector. It has been [available for production use since Java 15](https://openjdk.org/jeps/379) and has been designed to dramatically reduce garbage collection pause times, regardless of the heap size that is used. It can achieve these low pause times because most of the work is done before the GC pause, in a series of preparation steps. Shenandoah marks and compacts any heap objects eligible for garbage collection, while regular Java user threads are still running.

Java 24 introduces an extension to Shenandoah that maintains separate generations for young and old objects, allowing Shenandoah to collect young objects more frequently. This will result in a significant performance gain for applications running with generational Shenandoah, without sacrificing any of the valuable properties that the garbage collector is already known for.

The reason for handling young and old objects separately stems from the [weak generational hypothesis](https://docs.oracle.com/en/java/javase/17/gctuning/garbage-collector-implementation.html#GUID-71D796B3-CBAB-4D80-B5C3-2620E45F6E5D), which states that young objects tend to die young, while old objects tend to stick around. This means that collecting young objects requires fewer resources and yields more memory, while collecting old objects requires more resources and yields less memory. This is the reason we can improve the performance of applications that use Shenandoah by collecting young objects more frequently.

##### Running a Workload With Generational Shenandoah

Shenandoah used to behave in a non-generational way only. Running it required the following command-line configuration:

```bash
$ java ... -XX:+UseShenandoahGC
```

To run your workload with generational Shenandoah in Java 24, the following configuration is needed:

```bash
$ java ... -XX:+UseShenandoahGC -XX:+UnlockExperimentalVMOptions -XX:ShenandoahGCMode=generational
```

As you can see, generational Shenandoah has been introduced alongside non-generational Shenandoah. In a future release we can expect generational Shenandoah to become the default configuration.

##### More Information

For more information on this feature, read [JEP 404](https://openjdk.org/jeps/404).

#### JEP 450: Compact Object Headers (Experimental)

A Java object stored in the heap has metadata, which the HotSpot JVM stores in the object's header. Object headers are a fixed size, occupying between 96 and 128 bits, depending on how the JVM is configured. Since Java objects are often small ([typically 256 to 512 bits](https://wiki.openjdk.org/display/lilliput/Lilliput+Experiment+Results)), object headers can take up over 20% of live data. So reducing the size of object headers can significantly decrease memory footprint and garbage collection pressure. Project Lilliput experiments show a 10%-20% reduction in live data for real-world applications.

JEP 450 proposes to reduce the object header size to 64 bits, by merging the two parts that currently make up the object header: the *mark word* and the *class word*.

##### Legacy Object Header

The mark word comes first, has the size of a machine address, and contains:

```
Mark Word (normal):
 64                     39                              8    3  0
  [.......................HHHHHHHHHHHHHHHHHHHHHHHHHHHHHHH.AAAA.TT]
         (Unused)                      (Hash Code)     (GC Age)(Tag)
```

The class word comes after the mark word. It takes one of two shapes, depending on whether compressed class pointers are enabled:

```
Class Word (uncompressed):
64                                                               0
 [cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc]
                          (Class Pointer)

Class Word (compressed):
32                               0
 [CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC]
     (Compressed Class Pointer)
```

The class word is never overwritten, which means that an object's type information is always available, so no additional steps are required to check a type or invoke a method.

##### Compact Object Header

For compact object headers, the division between the mark and class word is removed:

```
Header (compact):
64                    42                             11   7   3  0
 [CCCCCCCCCCCCCCCCCCCCCCHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHVVVVAAAASTT]
 (Compressed Class Pointer)       (Hash Code)         /(GC Age)^(Tag)
                              (Valhalla-reserved bits) (Self Forwarded Tag)
```

As you can see, the size of the hash code does not change. 

> Note that four bits are reserved for future use by [Project Valhalla](https://openjdk.org/projects/valhalla/).

##### Future of This Feature

This experimental feature will have a broad impact on real-world applications. The code might have inefficiencies, bugs, and unanticipated non-bug behaviors. This feature must therefore be disabled by default and enabled only by explicit user request. We can expect the feature to become enabled by default in later releases and eventually the code for legacy object headers will be removed altogether.

##### Enabling Compact Object Headers

Compact object headers can be enabled as follows: 

```bash
$ java ... -XX:+UnlockExperimentalVMOptions -XX:+UseCompactObjectHeaders
```

##### More Information

For more information on this feature, read [JEP 450](https://openjdk.org/jeps/450).

#### JEP 475: Late Barrier Expansion for G1

The speed of Java applications has become increasingly important with the growing popularity of cloud-based Java deployments. An effective technique for speeding up Java applications is JIT compilation, but it incurs significant overhead in terms of processing time and memory usage. This is particularly noticeable with the C2 compiler which, through its use of early G1 barrier expansion, accounts for [up to 20% of the total overhead](https://robcasloz.github.io/blog/assets/c2-speed-results.html) incurred.

##### G1? C2? Early Barrier Expansion? Help Me Out Here!

*G1* has been Java's [default garbage collector since Java 9](https://openjdk.org/jeps/248). It's been designed to provide high performance and low pause times for applications with large heaps. It divides the heap into regions and prioritizes garbage collection in regions with the most garbage, hence the name "Garbage-First." G1 aims to achieve predictable pause times by performing most of its work concurrently with the application threads, minimizing the impact on application performance.

The *C2 compiler*, also known as the "HotSpot Server Compiler," is one of the Just-In-Time (JIT) compilers used by the HotSpot JVM in Java. It is designed to optimize and compile Java bytecode into highly optimized machine code at runtime, improving the performance of Java applications. The C2 compiler performs aggressive optimizations, such as inlining, loop unrolling, and escape analysis, to generate efficient native code for performance-critical parts of the application. It is typically used for long-running server applications where performance is crucial.

*Barrier expansion* is the process of inserting or generating additional code ('barriers') that manages memory and ensures the correctness of garbage collection. These barriers are typically inserted at specific points in the byte code, such as before or after memory access, to perform tasks like:

* *remembering writes*: keeping track of changes to objects, which helps the garbage collector identify which parts of the heap need to be scanned;
* *maintaining consistency*: ensuring that the program's view of memory remains consistent with the garbage collector's view, especially during concurrent garbage collection phases;
* *handling references*: managing references between objects, particularly when objects are moved during compaction or evacuation phases.

*Early barrier expansion* simply means that these barriers are inserted or generated early in the compilation process, whereas doing this *later* in the process (as the JEP proposes) would allow for more optimized placement and potentially reduce the overhead associated with these barriers. This can lead to improved performance and more efficient garbage collection.

##### More Information

For more information on this feature, read [JEP 475](https://openjdk.org/jeps/475). It has more details on the barrier expansion process, and how barrier expansion in the (early) *bytecode parsing* stage differs from barrier expansion in the (late) *code emission* stage.

#### JEP 483: Ahead-of-Time Class Loading & Linking

With features like dynamic class loading, dynamic reflection, dynamic compilation, annotation processing and native code optimization, the Java Platform is a highly dynamic one.
To be able to support these dynamic features, the JVM is forced to do a lot of work during startup, like:

* Scanning hundreds of JAR files on disk, while reading and parsing thousands of class files into memory;
* Loading the parsed class data into class objects and linking them together;
* Executing the static initializers of classes, which can create many objects and even perform I/O operations.

If the application uses a framework like Spring, then the startup-time discovery of `@Bean`, `@Configuration`, and related annotations will trigger yet more work.

The process described is performed on demand and optimized for quick execution, allowing many Java programs to start in milliseconds. 
However, larger server applications that utilize web frameworks and various libraries can take seconds or even minutes to launch.
Applications often repeat similar tasks during startup, such as scanning JAR files, loading classes, executing static initializers, and configuring application objects using reflection. 
To enhance startup speed, it's beneficial to perform some of these tasks proactively rather than waiting until they are needed. 
This approach aligns with the goals of [Project Leyden](https://openjdk.org/projects/leyden/), which strives to advance certain processes to an earlier stage.

##### Ahead-of-Time Cache

JEP 483, the first JEP out of Project Leyden, proposes to extend the JVM with an *ahead-of-time cache* to store classes after reading, parsing, loading and linking them.
A created cache for a specific application can be re-used in subsequent runs of that application to improve startup time.

Creating a cache takes two steps. 
First, you should run the application once in a training run, to record its AOT configuration (in this case into the file `app.aotconf`):

```bash
$ java -XX:AOTMode=record -XX:AOTConfiguration=app.aotconf -cp app.jar com.example.App ...
```

> Generally speaking, a production run is a good candidate for the training run, as training runs aim to capture application configuration and execution history. In cases where using a production run is impractical (due to activities or accessing databases), it's recommended to create a synthetic training run that closely resembles production runs, fully configuring itself and testing typical code paths. This can be done by adding a second main class, which invokes the production main class while using a temporary log directory, local network settings, and a mocked database if necessary. You might already have such a main class in the form of an integration test.

Second, use the configuration to create the cache, in the file `app.aot`:

```bash
$ java -XX:AOTMode=create -XX:AOTConfiguration=app.aotconf -XX:AOTCache=app.aot -cp app.jar
```

Subsequently, to run the application with the cache:

```bash
$ java -XX:AOTCache=app.aot -cp app.jar com.example.App ...
```

The AOT cache moves the tasks of reading, parsing, loading, and linking (typically performed just-in-time during program execution) to an earlier stage when the cache is created. 
As a result, the program starts up more quickly in the execution phase since its classes are readily accessible from the cache.

##### Performance Improvements of Up To 42%

To illustrate this, let's look at a short pragram that uses the Stream API and thus causes almost 600 JDK classes to be read, parsed, loaded, and linked:

```java
import java.util.*;
import java.util.stream.*;

public class HelloStream {
    public static void main(String[] args) {
        var words = List.of("hello", "fuzzy", "world");
        var greeting = words.stream()
            .filter(w -> !w.contains("z"))
            .collect(Collectors.joining(", "));
        System.out.println(greeting);  // hello, world
    }
}
```

This program runs in 0.031 seconds on JDK 23. 
After doing the small amount of additional work required to create an AOT cache it runs in in 0.018 seconds on JDK 24 — an improvement of 42%. The AOT cache occupies 11.4 megabytes.

For a representative server application, consider [Spring PetClinic](https://github.com/spring-projects/spring-petclinic) (v3.2.0). 
It loads and links about 21,000 classes at startup. 
It starts in 4.486 seconds on JDK 23 and in 2.604 seconds on JDK 24 when using an AOT cache — coincidentally also an improvement of 42%. Here the AOT cache occupies 130 megabytes.

##### More Information

For more information on this feature, read [JEP 483](https://openjdk.org/jeps/483).

#### JEP 491: Synchronize Virtual Threads Without Pinning

[Virtual threads](https://openjdk.org/jeps/444), available since Java 21, are lightweight threads that are scheduled by the JVM instead of by the operating system. Creating them and disposing of them is fast and cheap, and millions of them can be created within the same JVM.

A virtual thread goes through several stages in its lifetime:

* The virtual thread is *created* and linked to the code it should run during its lifetime.
* To actually run the code, the virtual thread is *mounted* on a platform thread, making that platform thread the *carrier* of the virtual thread.
* After running the code, the virtual thread is *unmounted* from its carrier and the platform thread is released, so the JDK's scheduler can mount a different virtual thread on it. Unmounting also happens when a virtual thread performs a blocking operation (such as I/O). When the blocking operation is ready to complete, the virtual thread is submitted back to the JDK's scheduler, which mounts it on a platform thread again to resume running code.

This means that virtual threads are mounted and unmounted frequently, without blocking any platform threads.

##### Pinning

But here's the catch: a virtual thread cannot unmount from its carrier when it runs code inside a `synchronized` block. Consider the class below, which is run by a virtual thread, tracking the number of customers in a store:

```java
class CustomerCounter {
    private final StoreRepository storeRepo;
    private int customerCount;

    CustomerCounter(StoreRepository storeRepo) {
        this.storeRepo = storeRepo;
        customerCount = 0;
    }

    synchronized void customerEnters() {
        if (customerCount < storeRepo.fetchCapacity()) {
            customerCount++;
        }
    }

    synchronized void customerExits() {
        customerCount--;
    }
}
```

If the `storeRepo.fetchCapacity()` method call blocks, it would be nice if the running virtual thread would unmount from its carrier, releasing a platform thread for other virtual threads to be mounted. 
But `customerEnters()` is `synchronized`, and because of this the JVM *pins* the virtual thread to its carrier, preventing it to be unmounted. 
The result is that both the virtual thread and the underlying OS thread are blocked, until the result from `fetchCapacity()` is available.

##### The Reason For Pinning

`synchronized` blocks and methods in Java rely on [monitors](https://en.wikipedia.org/wiki/Monitor_(synchronization)) to make sure they can be entered by a single thread at the same time. 
Before a thread can run a `synchronized` block, it has to acquire the monitor associated with the instance.
The JVM tracks ownership of these monitors on a *platform thread* level, not on a virtual thread.
Given that information, imagine for a moment that *pinning* didn't exist.
Then in theory, virtual thread #1 could unmount in the middle of a `synchronized` block, and virtual thread #2 could be mounted on the same platform thread, continuing that same `synchronized` block because the carrier thread is the same and still holds the object's monitor.
Understandably, the JVM actively prevents this situation!

##### Overcoming Pinning

So pinning *does* have a purpose, but frequent pinning for long durations can harm scalability.
Because of this, many libraries have switched to using the more flexible [`java.util.concurrent` locks](https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/util/concurrent/locks/package-summary.html) instead, which do not pin virtual threads.
But this is a workaround at best, and that's why JEP 491 proposes to overcome virtual thread pinning.

From Java 24 on, virtual threads can acquire, hold and release monitors, regardless of their carriers.
This also means that switching to different locking mechanisms because of thread pinning is no longer necessary.
Both approaches will perform equally well with virtual threads from now on.

##### Remaining Pinning Cases

One of the few remaining situations in which a virtual thread will still be pinned, is when it calls native code, which returns to Java code that performs a blocking operation.
In cases like this, the JDK Flight Recorder will record a [`jdk.VirtualThreadPinned`](https://openjdk.org/jeps/444#JDK-Flight-Recorder-JFR) event, should you want to keep track of these situations.

##### More Information

For more information on this feature, read [JEP 491](https://openjdk.org/jeps/491).

### Security Libs

Java 24 introduces three new features that are part of the Security Libs:

* Key Derivation Function API (Preview)
* Quantum-Resistant Module-Lattice-Based Key Encapsulation Mechanism
* Quantum-Resistant Module-Lattice-Based Digital Signature Algorithm

#### JEP 478: Key Derivation Function API (Preview)

As the field of quantum computing advances, traditional cryptographic algorithms are becoming more susceptible to practical attacks. Thus, it is essential for the Java Platform to incorporate Post-Quantum Cryptography (PQC), which can withstand such threats. Java's long-term goal is to eventually implement Hybrid Public Key Encryption (HPKE), facilitating a seamless transition to quantum-resistant encryption methods. The [KEM API (JEP 452)](https://hanno.codes/2023/09/19/java-21-release-day/#jep-452-key-encapsulation-mechanism-api), included in JDK 21, serves as one component of HPKE and marks Java's initial move towards HPKE and readiness for post-quantum challenges. This JEP proposes an additional component of HPKE as a next step in this direction: an API for [Key Derivation Functions](https://en.wikipedia.org/wiki/Key_derivation_function) (KDFs). 

KDF's are cryptographic algorithms for deriving additional keys from a secret key and other data. A KDF allows keys to be created in a manner that is both secure and reproducible by two parties sharing knowledge of the inputs. Deriving keys is similar to hashing passwords. A KDF employs a keyed hash along with extra entropy from its other inputs to either derive new key material or safely expand existing values into a larger quantity of key material.

##### Operations

A key derivation function has two fundamental operations:

* *instantiation and initialization*, creating a KDF and initializing it with the appropriate parameters;
* *derivation*, accepting key material and other optional inputs as well as parameters to describe the output, and generating the derived key or data.

A new class, [`javax.crypto.KDF`](https://cr.openjdk.org/~kdriver/KDF-JEP/javadoc/java.base/javax/crypto/KDF.html) represents key derivation functions.

##### Code Example

To get an idea of how to use this API, see the code example below (taken from the JEP):

```java
// Create a KDF object for the specified algorithm
KDF hkdf = KDF.getInstance("HKDF-SHA256"); 

// Create an ExtractExpand parameter specification
AlgorithmParameterSpec params =
    HKDFParameterSpec.ofExtract()
                     .addIKM(initialKeyMaterial)
                     .addSalt(salt).thenExpand(info, 32);

// Derive a 32-byte AES key
SecretKey key = hkdf.deriveKey("AES", params);

// Additional deriveKey calls can be made with the same KDF object
```

##### Preview Warning

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

##### More Information

For more information on this feature, read [JEP 478](https://openjdk.org/jeps/478).

#### JEP 496: Quantum-Resistant Module-Lattice-Based Key Encapsulation Mechanism

We mentioned quantum computing advancements before in this article, and with good reason. Public-key based algorithms like Rivest-Shamir-Adleman (RSA) and Diffie-Hellman are more and more at risk of practical quantum computing attacks, but they are still in use by the Java Platform (for things like digitally signing JAR files or establishing secure network connections through TLS). To address this issue, [quantum-resistant](https://en.wikipedia.org/wiki/Post-quantum_cryptography) cryptographic algorithms have been invented, and this JEP introduces an implementation of one of those algorithms: the [Module-Lattice-Based Key-Encapsulation Mechanism](https://csrc.nist.gov/pubs/fips/203/final) (ML-KEM).

##### KEM Components

As described in [JEP 452](https://hanno.codes/2023/09/19/java-21-release-day/#jep-452-key-encapsulation-mechanism-api), any key encapsulation mechanism needs the following components:

* a *key pair generation function* that returns a key pair containing a public key and a private key. 
* a *key encapsulation function*, called by the sender, that takes the receiver's public key and an encryption option; it returns a secret key and a _key encapsulation message_. The sender sends the key encapsulation message to the receiver.
* a *key decapsulation function*, called by the receiver, that takes the receiver's private key and the received key encapsulation message; it returns the secret key.

The ML-KEM algorithm provides an implementation of the `KeyPairGenerator` API to generate ML-KEM key pairs. To encapsulate and decapsulate keys, an implementation of the KEM API is provided that negotiates shared secret keys based on an ML-KEM key pair. Also provided is an implementation of the `KeyFactory` API, that converts ML-KEM keys to and from their encodings.

Furthermore, a new standard algorithm family name ("ML-KEM") will be defined in the [Java Security Standard Algorithm Names Specification](https://docs.oracle.com/en/java/javase/23/docs/specs/security/standard-names.html). The algorithm comes with three parameter sets (increasing in security, decreasing in performance): 
"ML-KEM-512", "ML-KEM-768", and "ML-KEM-1024". They are represented by the new `NamedParameterSpec` constants `ML_KEM_512`, `ML_KEM_768` and `ML_KEM_1024`.

##### Generating ML-KEM Key Pairs

You can generate an ML-KEM key pair in one of three ways:

```java
KeyPairGenerator generator = KeyPairGenerator.getInstance("ML-KEM");
generator.initialize(NamedParameterSpec.ML_KEM_512);
KeyPair keyPair = generator.generateKeyPair(); // an ML-KEM-512 key pair
```

```java
KeyPairGenerator generator = KeyPairGenerator.getInstance("ML-KEM");
KeyPair keyPair = generator.generateKeyPair(); // an ML-KEM-768 key pair by default
```

```java
KeyPairGenerator generator = KeyPairGenerator.getInstance("ML-KEM-1024");
KeyPair keyPair = generator.generateKeyPair(); // an ML-KEM-1024 key pair
```

##### keytool

The `keytool` command will support generating ML-KEM key pairs and certificates; 

```bash
$ keytool -keystore ks -storepass changeit -genkeypair -alias ec \
          -keyalg ec -dname CN=ec -ext bc
$ keytool -keystore ks -storepass changeit -genkeypair -alias mlkem \
          -keyalg ML-KEM -groupname ML-KEM-768 -dname CN=ML-KEM -signer ec
```

The parameter-set name (`ML-KEM-768`) can also be provided directly with the `-keyalg` option:

```bash
$ keytool -keystore ks -storepass changeit -genkeypair -alias mlkem2 \
          -keyalg ML-KEM-768 -dname CN=ML-KEM2 -signer ec
```

##### Encapsulating and Decapsulating Keys

You can use the ML-KEM `KEM` implementation to negotiate a shared secret key.

For example, a sender can call the encapsulation function to get a secret key and a key encapsulation message:

```java
KEM ks = KEM.getInstance("ML-KEM");
KEM.Encapsulator enc = ks.newEncapsulator(publicKey);
KEM.Encapsulated encap = enc.encapsulate();
byte[] msg = encap.encapsulation();     // send this to receiver
SecretKey sks = encap.key();
```

A receiver can then call the decapsulation function to recover the secret key from the key encapsulation message sent by the sender:

```java
byte[] msg = ...;                       // received from sender
KEM kr = KEM.getInstance("ML-KEM");
KEM.Decapsulator dec = kr.newDecapsulator(privateKey);
SecretKey skr = dec.decapsulate(msg);
```

Both `sks` and `skr` now contain the same key material, which is known only to the sender and the receiver.

##### More Information

For more information on this feature, read [JEP 496](https://openjdk.org/jeps/496).

#### JEP 497: Quantum-Resistant Module-Lattice-Based Digital Signature Algorithm

For the exact same reason as the previous JEP, this one introduces an implementation of a different quantum-resistant algorithm: the [Module-Lattice-Based Digital Signature Algorithm](https://csrc.nist.gov/pubs/fips/204/final) (ML-DSA).

The ML-DSA algorithm provides an implementation of both the `KeyPairGenerator` API (to generate ML-DSA key pairs) and the `Signature` API (to sign and verify ML-DSA signatures). Also provided is an implementation of the `KeyFactory` API, that converts ML-DSA keys to and from their encodings.

Furthermore, a new standard algorithm family name ("ML-DSA") will be defined in the [Java Security Standard Algorithm Names Specification](https://docs.oracle.com/en/java/javase/23/docs/specs/security/standard-names.html). The algorithm comes with three parameter sets (increasing in security, decreasing in performance): 
"ML-DSA-44", "ML-DSA-65", and "ML-DSA-87". They are represented by the new `NamedParameterSpec` constants `ML_DSA_44`, `ML_DSA_65` and `ML_DSA_87`.

##### Generating ML-DSA Key Pairs

You can generate an ML-DSA key pair in one of three ways:

```java
KeyPairGenerator generator = KeyPairGenerator.getInstance("ML-DSA");
generator.initialize(NamedParameterSpec.ML_DSA_44);
KeyPair keyPair = generator.generateKeyPair(); // an ML-DSA-44 key pair
```

```java
KeyPairGenerator generator = KeyPairGenerator.getInstance("ML-DSA");
KeyPair keyPair = generator.generateKeyPair(); // an ML-DSA-65 key pair by default
```

```java
KeyPairGenerator generator = KeyPairGenerator.getInstance("ML-DSA-87");
KeyPair keyPair = generator.generateKeyPair(); // an ML-DSA-87 key pair
```

##### keytool

The `keytool` command will support generating ML-DSA key pairs and certificates; 

```bash
$ keytool -keystore ks -storepass changeit -genkeypair -alias mldsa \
          -keyalg ML-DSA -groupname ML-DSA-65 -dname CN=ML-DSA

$ keytool -keystore ks -storepass changeit -genkeypair -alias mldsa \
          -keyalg ML-DSA-65 -dname CN=ML-DSA2
```

The parameter-set name (`ML-DSA-65`) can also be provided directly with the `-keyalg` option:

```bash
$ keytool -keystore ks -storepass changeit -genkeypair -alias mldsa \
          -keyalg ML-DSA-65 -dname CN=ML-DSA2
```

##### Signing with ML-DSA Keys

You can use the ML-DSA Signature implementation to sign and verify ML-DSA signatures.

For example, to sign a message using a private key:

```java
byte[] msg = ...;
Signature ss = Signature.getInstance("ML-DSA");
ss.initSign(privateKey);
ss.update(msg);
byte[] sig = ss.sign();
```

To verify a signature with a public key:

```java
byte[] msg = ...;
byte[] sig = ...;
Signature sv = Signature.getInstance("ML-DSA");
sv.initVerify(publicKey);
sv.update(msg);
boolean verified = sv.verify(sig);
```

##### More Information

For more information on this feature, read [JEP 497](https://openjdk.org/jeps/497).

### Tools

Java 24 contains a single feature that is part of the JLink tool:

* Linking Run-Time Images Without JMODs

#### JEP 493: Linking Run-Time Images Without JMODs

In cloud environments, container images that include an installed JDK are frequently copied over the network from container registries. 
Reducing the size of the JDK would improve the efficiency of these operations.
This is why the [`jlink`](https://dev.java/learn/jlink/) tool can now create custom run-time images without using the JDK's JMOD files.
This allows users to link a run-time image from modules, regardless of their source (be it JMOD files, modular JARs or part of a run-time image linked previously).

##### Redundancy

An installed JDK consists of a *run-time image* and a set of *packaged modules* in the JMOD format, for each module in the run-time image.
`jlink` uses the JMOD files when creating custom run-time images.
In fact, the run-time image in the JDK is a result from this very process.
That means that every resource in the JDK's run-time image is also present in one of the JMOD files, which makes an installed JDK suffer from *redundancy*.
The JMOD files account for about 25% of the JDK's total size.
If `jlink` could be changed to extract resources from the run-time image itself, we could dramatically reduce the size of the JDK by simply omitting these JMOD files.

##### Build a JDK With `--enable-linkable-runtime`

The option `--enable-linkable-runtime` builds a JDK whose `jlink` tool can create run-time images without using JMOD files.
The resulting JDK will have no `jmods` directory, which means it's about 25% smaller than before.

```bash
$ configure [ ... other options ... ] --enable-linkable-runtime
$ make images
```

The `jlink` tool in any JDK build can consume both JMOD files and modular JAR files. 
In addition, in JDK builds with this feature enabled, `jlink` can consume modules from the run-time image of which it is part. 
The `--help` output of `jlink` shows whether it has this capability:

```bash
$ jlink --help
Usage: jlink <options> --module-path <modulepath> --add-modules <module>[,<module>...]
...
Capabilities:
      Linking from run-time image enabled
```

So this new capability can be enabled only when building a JDK.
This also means that any `jlink` invocation that wants to make use of it doesn't need any additional options — it can remain exactly the same.
From Java 24 on, `jlink` will use JMOD files if it can find them on the module path.
It will consume modules from the run-time image only if the module `java.base` is not found on the module path. 
Any other modules must still be specified to `jlink` via the `--module-path` option.

##### Not enabled by default

This feature is currently not enabled by default, so the default JDK build configuration will remain as it is today.
The resulting JDK will contain JMOD files like before and its `jlink` tool will not be able to operate without them. 
Whether the JDK build that you get from your preferred vendor contains this feature is up to that vendor.
Note that the JEP also states that the feature may be enabled by default in a future release.

##### More Information

For more information on this feature, read [JEP 493](https://openjdk.org/jeps/493).

## Repreviews and finalizations

Now it's time to take a look at a few features that might already be familiar to you, because they were introduced in a previous version of Java. They have been repreviewed (or finalized) in Java 24, with only minor changes compared to Java 23 in most cases. Therefore, to avoid a very lengthy article, we'll outline these changes and link to a previous article for a full feature description, should you wish to refresh your memory.

### JEP 484: Class-File API

Java frameworks often use bytecode transformation to add functionality, relying on libraries like [ASM](https://asm.ow2.io/) or [Javassist](https://www.javassist.org/). However, the JDK's six-month release cycle can outpace these libraries, causing compatibility issues. JEP 484 addresses this by proposing a standard class-file API that evolves with the JDK, ensuring up-to-date class file processing.

#### What's Different From Java 23?

A few minor things changed to the API, based on feedback from the second preview stage. These changes mainly include the removal of a few fields, methods and interfaces, and the renaming of various enum values, fields and methods. The JEP includes [a detailed list](https://openjdk.org/jeps/484#Changes) on these changes.

#### More Information

For more information on this feature, read [JEP 484](https://openjdk.org/jeps/484) or the [full feature description](https://hanno.codes/2024/09/17/java-23-has-arrived/#jep-466-class-file-api-second-preview) from a previous article.

### JEP 485: Stream Gatherers

The Stream API offers a relatively diverse but predetermined range of intermediate operations, including mapping, filtering, sorting and more. JEP 485 introduces *stream gatherers*, which allow developers to define their own custom intermediate stream operations, so they can transform streams in their own preferred ways.

#### What's Different From Java 23?

Nothing was changed, apart from the fact that the API is now finalized in Java 24.

#### More Information

For more information on this feature, read [JEP 485](https://openjdk.org/jeps/485) or the [full feature description](https://hanno.codes/2024/09/17/java-23-has-arrived/#jep-473-stream-gatherers-second-preview) from a previous article.

### JEP 487: Scoped Values (Fourth Preview)

*Scoped values* enable the sharing of immutable data within and across threads. 
They are preferred to thread-local variables, especially when using a large number of (virtual) threads.

#### What's Different From Java 23?

A single change was made to the API compared to Java 23:

* The `callWhere` and `runWhere` methods were removed from the `ScopedValue` class, which means that the entire API is now fluent. The exact same behavior can be obtained by chaining `ScopedValue.where()` with `run(Runnable)` or `call(Callable)`.

#### Preview Warning

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, read [JEP 487](https://openjdk.org/jeps/487) or the [full feature description](https://hanno.codes/2024/09/17/java-23-has-arrived/#jep-481-scoped-values-third-preview) from a previous article.

### JEP 488: Primitive Types in Patterns, instanceof and switch (Second Preview)

Pattern matching now supports primitive types in all pattern contexts. On top of that, the `instanceof` and `switch` constructs have been extended to also work with all primitive types.

#### What's Different From Java 23?

Compared to the preview version of this feature in Java 22, nothing was changed or added. JEP 488 simply exists to gather more feedback from users.

#### Preview Warning

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, read [JEP 488](https://openjdk.org/jeps/488) or the [full feature description](https://hanno.codes/2024/09/17/java-23-has-arrived/#jep-455-primitive-types-in-patterns-instanceof-and-switch-preview) from a previous article.

### JEP 489: Vector API (Ninth Incubator)

The Vector API makes it possible to express vector computations that reliably compile at runtime to optimal vector instructions. 
This means that these computations will significantly outperform equivalent scalar computations on the supported CPU architectures (x64 and AArch64).

#### What's Different From Java 23?

The following changes were made to the Vector API compared to Java 23:

* A new variation of the `selectFrom` cross-lane operation now accepts two input vectors, which serve as a lookup table;
* The `selectFrom` and `rearrange` cross-lane operations now wrap indexes rather than check for out-of-bounds indexes, making these operations significantly faster.
* Transcendental and trigonometric lanewise operations on ARM and RISC-V are now implemented via intrinsics which call out to the [SIMD Library for Evaluating Elementary Functions (SLEEF)](https://sleef.org/).
* The new value-based class [Float16](https://download.java.net/java/early_access/jdk24/docs/api/jdk.incubator.vector/jdk/incubator/vector/Float16.html) represents 16-bit floating-point numbers in the [IEEE 754 binary16](https://en.wikipedia.org/wiki/Half-precision_floating-point_format) format.
* The arithmetic integral lanewise operations now include saturating unsigned addition and subtraction, saturating signed addition and subtraction, and unsigned maximum and minimum.

The Vector API will keep incubating until necessary features of Project Valhalla become available as preview features. When that happens, the Vector API will be adapted to use them, and it will be promoted from incubation to preview.

#### More Information

For more information on this feature, read [JEP 489](https://openjdk.org/jeps/489) or the [full feature description](https://hanno.codes/2024/09/17/java-23-has-arrived/#jep-469-vector-api-eighth-incubator) from a previous article.

### JEP 492: Flexible Constructor Bodies (Third Preview)

Flexible constructor bodies allow statements to appear before an explicit constructor invocation, like `super(..)` or `this(..)`. The statements cannot reference the instance under construction, but they can initialize its fields. Initializing fields before invoking another constructor makes a class more reliable when methods are overridden.

#### Preview Warning

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### What's Different From Java 23?

Compared to the preview version of this feature in Java 23, no significant changes were made. JEP 492 simply exists to gather more feedback from users.

#### More Information

For more information on this feature, read [JEP 492](https://openjdk.org/jeps/492) or the [full feature description](https://hanno.codes/2024/09/17/java-23-has-arrived/#jep-482-flexible-constructor-bodies-second-preview) from a previous article.

### JEP 494: Module Import Declarations (Second Preview)

Module import declarations import all of the public top-level classes and interfaces in the packages exported by that module. They are a shorter alternative for listing many imports that originate from the same root package.

#### What's Different From Java 23?

This feature is repreviewed in Java 24, with two additions:

* The restriction that no module is able to declare a transitive dependency on the `java.base` module is lifted, and the declaration of the `java.se` module to transitively require the `java.base` module is revised. With these changes, importing the `java.se` module will import the entire Java SE API on demand.
* Type-import-on-demand declarations to shadow module import declarations are now allowed.

#### More Information

For more information on this feature, read [JEP 494](https://openjdk.org/jeps/494) or the [full feature description](https://hanno.codes/2024/09/17/java-23-has-arrived/#jep-476-module-import-declarations-preview) from a previous article.

### JEP 495: Simple Source Files and Instance Main Methods (Fourth Preview)

Simple source files allow developers to write Java programs without the need to explicitly declare a class. They can contain 'instance main methods': a shorter form of the classic `main()` method without requiring program arguments or imports. These two features simplify the process of writing small programs and scripts by reducing boilerplate code.

#### What's Different From Java 23?

The two features in this JEP didn't change in Java 24; however, the feature that used to be known as 'implicitly declared classes' was renamed to 'simple source files'.

#### More Information

For more information on this feature, read [JEP 495](https://openjdk.org/jeps/495) or the [full feature description](https://hanno.codes/2024/09/17/java-23-has-arrived/#jep-477-implicitly-declared-classes-and-instance-main-methods-third-preview) from a previous article.

### JEP 499: Structured Concurrency (Fourth Preview)

*Structured concurrency* treats groups of related tasks running in different threads as a single unit of work, thereby streamlining error handling and cancellation, improving reliability, and enhancing observability.

#### What's Different From Java 23?

Nothing was changed, the API is re-previewed in Java 24 to give more time for feedback from real world usage.

However, significant API changes are scheduled to appear in a future Java release.
A `StructuredTaskScope` will then be opened via static factory methods rather than through public constructors. The zero-parameter `open()` factory method will cover the common case by creating a `StructuredTaskScope` that waits for all subtasks to succeed or any subtask to fail. Other policies and outcomes can be implemented by providing an appropriate [`Joiner`](https://download.java.net/java/early_access/loom/docs/api/java.base/java/util/concurrent/StructuredTaskScope.Joiner.html) to one of the richer `open(Joiner)` factory methods.

If you're interested in the future of this feature, [JEP draft #8340343](https://openjdk.org/jeps/8340343) has more details on these upcoming API changes.

#### More Information

If you prefer to get more information on the current state of this feature, then read [JEP 499](https://openjdk.org/jeps/499) or the [full feature description](https://hanno.codes/2024/09/17/java-23-has-arrived/#jep-480-structured-concurrency-third-preview) from a previous article.

## Deprecations & Restrictions

Java 24 also deprecates a few older features that weren't used that much and restricts a few other features that came with certain risks. Let's see which ones were involved in this effort to improve stability.

### JEP 472: Prepare to Restrict the Use of JNI

The Java Native Interface has been the factory standard of invoking foreign functions (outside of the JVM but on the same machine) for many years now. In Java 22 a more modern approach became available: the [Foreign Function & Memory API](https://hanno.codes/2024/03/19/java-22-is-here/#jep-454-foreign-function--memory-api). Although the new FFM API is the preferred alternative to JNI, that doesn't mean that JNI will be phased out. On the contrary, the Java language designers want to make sure that migrating from one to the other can be done easily. To make that happen, this JEP introduces certain warnings to JNI to mirror the warnings that the FFM API already produces. 

To be more precise, calling native code via JNI requires you to load a native library first and link a Java construct to a function in that library. JEP 472 will add the [restrictions](https://openjdk.org/jeps/454#Safety) that the FFM API already imposes on these loading and linking steps to JNI also. All warnings of this type aim to prepare developers for a future release that ensures [integrity by default](https://openjdk.org/jeps/8305968) by uniformly restricting JNI and the FFM API.

#### More Information

For more information on this feature, read [JEP 472](https://openjdk.org/jeps/472).

### JEP 486: Permanently Disable the Security Manager

The Security Manager has not been the primary method for securing client-side Java code for many years and is seldom used for server-side code. It is also costly to maintain. Consequently, it was deprecated for removal in Java 17 via [JEP 411](https://openjdk.org/jeps/411). JEP 486 takes the next logical step: developers are now prevented from enabling the Security Manager at all. The Java Language Designers feel this is a safe step to take, as the deprecation in Java 17 hardly had any impact. The Security Manager API will be removed in a future release.

#### OK, But What Exactly *Is* the Security Manager?

That is a fair question, because it has rarely been used. The Security Manager, a feature since Java's first release, operates on *the principle of least privilege*: code is untrusted by default, so it cannot access resources such as the filesystem or the network. This restricts code access to resources unless explicitly granted permission. Despite its theoretical benefits, its complexity has led to it being rarely used and has always been disabled by default. The least-privilege model adds significant complexity to Java Platform libraries, requiring over 1,000 methods to check permissions and over 1,200 methods to elevate privileges when the Security Manager is enabled.

#### More Information

For more information on this change, read [JEP 486](https://openjdk.org/jeps/486).

### JEP 490: ZGC: Remove the Non-Generational Mode

The Z Garbage Collector (ZGC) is a scalable, low-latency garbage collector. It has been [available for production use since Java 15](https://openjdk.org/jeps/377) and has been designed to keep pause times consistent and short, even for very large heaps. It uses techniques like region-based memory management and compaction to achieve this.

Java 21 introduced [an extension to ZGC](https://openjdk.org/jeps/439) that maintains separate *generations* for young and old objects, allowing ZGC to collect young objects (which tend to die young) more frequently. This will result in a significant performance gain for applications running with generational ZGC, without sacrificing any of the valuable properties that the Z garbage collector is already known for. Java 23 made generational mode the default mode for ZGC and deprecated non-generational mode. Java 24 removes non-generational mode altogether.

#### How Do the ZGC Command-Line Options Work After This Change?

In Java 23, the following command would use generational mode by default:

```bash
$ java -XX:+UseZGC ...
```

This behavior is still the same in Java 24. Also, in Java 24, if you would run Java with the now-obsolete option `ZGenerational`...

```bash
$ java -XX:+UseZGC -XX:-ZGenerational ...
```

...an obsolete-option warning will be printed.

#### More Information

For more information on this feature, read [JEP 490](https://openjdk.org/jeps/490).

### JEP 498: Warn upon Use of Memory-Access Methods in sun.misc.Unsafe

The `sun.misc.Unsafe` class contains 87 methods to perform low-level operations, such as accessing off-heap memory.
The class is aptly named: using its methods without performing the necessary safety checks can lead to undefined behaviour and to the JVM crashing.
They were meant exclusively for use within the JDK, but back in 2002 when the class was introduced the [module system](https://openjdk.org/projects/jigsaw/) wasn't around yet and so there was no way to prevent the class from being used outside the JDK.
Thus, the memory-access methods in `sun.misc.Unsafe` became a valuable tool for library developers seeking greater power and performance than what standard APIs could provide.

Two standard APIs have emerged in recent years that are far better alternatives to these problematic methods:

* [`java.lang.invoke.VarHandle`](https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/invoke/VarHandle.html), to manipulate on-heap memory safely and efficiently;
* [`java.lang.foreign.MemorySegment`](https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/foreign/MemorySegment.html), to access off-heap memory safely and efficiently.

Given the situation, Java 23 already deprecated the memory-access methods, which led to Java generating *compile-time* warnings whenever these methods were used. JEP 490 extends this behavior by also generating *run-time* warnings for any use of these methods.

JDK 24 will, by default, issue a warning on the first occasion that any memory-access method is used, whether directly or via reflection. That is, it will issue at most one warning regardless of which memory-access methods are used and how many times any particular method is used. This will alert application developers and users to the forthcoming removal of the methods, and the need to upgrade libraries. An example of the warning is:

```
WARNING: A terminally deprecated method in sun.misc.Unsafe has been called
WARNING: sun.misc.Unsafe::setMemory has been called by com.foo.bar.Server (file:/tmp/foobarserver/thing.jar)
WARNING: Please consider reporting this to the maintainers of com.foo.bar.Server
WARNING: sun.misc.Unsafe::setMemory will be removed in a future release
```

A future release of Java will start throwing exceptions in these situations. In an even later Java release the methods will be removed entirely. According to the JEP text the entire process won't be completed until after the release of JDK 26, giving developers ample time to adjust to the new situation.

#### More Information

For more information on this feature, see [JEP 498](https://openjdk.org/jeps/498). It has more details on the targeted methods, their alternatives from `VarHandle` and `MemorySegment` and how to configure the deprecation warnings with the new command-line option `--sun-misc-unsafe-memory-access` (to promote the warnings to `UnsupportedOperationException`s already, for example). It also provides a few migration examples. 

### JEP 501: Deprecate the 32-bit x86 Port for Removal

Supporting multiple platforms has been the focus of the Java ecosystem since the beginning. 
But older platforms cannot be supported indefinitely, and that is one of the reasons why the 32-bit x86 (Linux) port is now scheduled for removal.
The effort that was required to maintain this port exceeded its advantages. Keeping it up-to-date with new features like Loom, the Foreign Function & Memory API (FFM), the Vector API, and late GC barrier expansion represented a significant cost, so it is now deprecated.oe

Configuring a 32-bit x86 build will now fail on JDK 24.
This error can be suppressed by using the new build configuration option `--enable-deprecated-ports=yes`.
This means 32-bit users can still use JDK 24; however a future release will actually remove the support, and by that time the affected users are expected to have migrated to 64-bit JVMs.

#### More Information

For more information on this deprecation, read [JEP 501](https://openjdk.org/jeps/501).

### JEP 479: Remove the Windows 32-bit x86 Port

This JEP removes the Windows 32-bit x86 port, which was to be expected after [its deprecation in Java 21](https://hanno.codes/2023/09/19/java-21-release-day/#jep-449-deprecate-the-windows-32-bit-x86-port-for-removal). This is due to the fact that Windows 10, the last Windows operating system to support 32-bit operation, [will reach end-of-life in October 2025](https://learn.microsoft.com/lifecycle/products/windows-10-home-and-pro). On top of that, the implementation of virtual threads on Windows 32-bit x86 is rudimentary to say the least: it uses a single platform thread for each virtual thread, effectively rendering the feature useless on this platform. So it's time to say goodbye to this port!

#### More Information

For more information on this removal, read [JEP 479](https://openjdk.org/jeps/479).

## Final thoughts

And that concludes our discussion of the 24 JEP's that come with Java 24. But that's not even all that's new: [many other updates](https://jdk.java.net/24/release-notes) were included in this release, including various performance, stability and security updates. So what are you waiting for? Time to take this brand-new Java release for a spin!
