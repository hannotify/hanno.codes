---
layout: post
title: Java 24 Rolls Out Today! Find Out Why It's Aptly Named
date: 18-03-2025 04:30:00 +0200
header:
  teaser: /assets/images/blog/basketball-24.jpg
excerpt: Java 24 TODO - excerpt
tags: 
- java
---

TODO - write intro
TODO - explain article structure

TODO - replace with a smaller version of this image

![Basketball players hugging during game - one of them wears a jersey with number '24' at the back](/assets/images/blog/basketball-24.jpg)
> Photo by Royy Nguyen, from <a href="https://www.pexels.com/photo/basketball-players-hugging-during-game-in-gym-5303477/">Pexels</a>

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

Java 24 introduces an extension to Shenandoah that maintains separate generations for young and old objects, allowing Shenandoah to collect young objects (which tend to die young) more frequently. This will result in a significant performance gain for applications running with generational Shenandoah, without sacrificing any of the valuable properties that the garbage collector is already known for.

The reason for handling young and old objects separately stems from the [weak generational hypothesis](https://docs.oracle.com/en/java/javase/17/gctuning/garbage-collector-implementation.html#GUID-71D796B3-CBAB-4D80-B5C3-2620E45F6E5D), which states that young objects tend to die young, while old objects tend to stick around. This means that collecting young objects requires fewer resources and yields more memory, while collecting old objects requires more resources and yields less memory. This is the reason we can improve the performance of applications that use Shenandoah by collecting young objects more frequently.

##### Running a Workload With Generational Shenandoah

Shenandoah used to be able to behave in a non-generational way only. Running it required the following command-line configuration:

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

As you can see, the size of the hash code does not change. Four bits are reserved for future use by [Project Valhalla](https://openjdk.org/projects/valhalla/).

##### Future of This Feature

This experimental feature will have a broad impact on real-world applications. The code might have inefficiencies, bugs, and unanticipated non-bug behaviors. This feature must therefore be disabled by default and enabled only by explicit user request. We can expect the feature to become enabled by default in later releases and eventually the code for legacy object headers will be removed altogether.

##### Enabling Compact Object Headers

Compact object headers is an experimental feature and therefore disabled by default. They can be enabled with: 

```bash
$ java ... -XX:+UnlockExperimentalVMOptions -XX:+UseCompactObjectHeaders
```

##### More Information

For more information on this feature, read [JEP 450](https://openjdk.org/jeps/450).

#### JEP 475: Late Barrier Expansion for G1

The speed of Java applications has become increasingly important with the growing popularity of cloud-based Java deployments. An effective technique for speeding up Java applications is JIT compilation, but it incurs significant overhead in terms of processing time and memory usage. This is particularly noticeable with the C2 compiler which, through its use of early G1 barrier expansion, accounts for [up to 20% of the total overhead](https://robcasloz.github.io/blog/assets/c2-speed-results.html) incurred.

##### G1? C2? Early Barrier Expansion? Help Me Out Here!

*G1* has been Java's [default garbage collector since Java 9](https://openjdk.org/jeps/248). It is a garbage collector designed to provide high performance and low pause times for applications with large heaps. It divides the heap into regions and prioritizes garbage collection in regions with the most garbage, hence the name "Garbage-First." G1 aims to achieve predictable pause times by performing most of its work concurrently with the application threads, minimizing the impact on application performance.

The *C2 compiler*, also known as the "HotSpot Server Compiler," is one of the Just-In-Time (JIT) compilers used by the HotSpot JVM in Java. It is designed to optimize and compile Java bytecode into highly optimized machine code at runtime, improving the performance of Java applications. The C2 compiler performs aggressive optimizations, such as inlining, loop unrolling, and escape analysis, to generate efficient native code for performance-critical parts of the application. It is typically used for long-running server applications where performance is crucial.

*Expanding barriers* refers to the process of inserting or generating additional code ('barriers') that manage memory and ensure the correctness of garbage collection. These barriers are typically inserted at specific points in the byte code, such as before or after memory access, to perform tasks like:

* *remembering writes*: keeping track of changes to objects, which helps the garbage collector identify which parts of the heap need to be scanned.
* *maintaining consistency*: ensuring that the program's view of memory remains consistent with the garbage collector's view, especially during concurrent garbage collection phases.
* *handling references*: managing references between objects, particularly when objects are moved during compaction or evacuation phases.

*Early barrier expansion* simply means that these barriers are inserted or generated earlier in the compilation process, whereas doing this *later* in the process (as the JEP proposes) would allow for more optimized placement and potentially reduce the overhead associated with these barriers. This can lead to improved performance and more efficient garbage collection.

TODO: explain in general terms how early barrier expansion actually improves performance.

##### More Information

For more information on this feature, read [JEP 475](https://openjdk.org/jeps/475). It has more details on TODO.

#### JEP 483: Ahead-of-Time Class Loading & Linking

TODO

##### More Information

For more information on this feature, read [JEP TODO](https://openjdk.org/jeps/todo).

#### JEP 491: Synchronize Virtual Threads Without Pinning

TODO

##### More Information

For more information on this feature, read [JEP TODO](https://openjdk.org/jeps/todo).

### Security Libs

Java 24 introduces three new features that are part of the Security Libs:

* Key Derivation Function API (Preview)
* Quantum-Resistant Module-Lattice-Based Key Encapsulation Mechanism
* Quantum-Resistant Module-Lattice-Based Digital Signature Algorithm

#### JEP 478: Key Derivation Function API (Preview)

As quantum computing advances, traditional cryptographic algorithms are becoming more susceptible to practical attacks. Thus, it is essential for the Java Platform to incorporate Post-Quantum Cryptography (PQC), which can withstand such threats. Java's long-term goal is to eventually implement Hybrid Public Key Encryption (HPKE), facilitating a seamless transition to quantum-resistant encryption methods. The [KEM API (JEP 452)](/2023/09/19/java-21-release-day/#jep-452-key-encapsulation-mechanism-api), included in JDK 21, serves as one component of HPKE and marks Java's initial move towards HPKE and readiness for post-quantum challenges. This JEP proposes an additional component of HPKE as a next step in this direction: an API for [Key Derivation Functions](https://en.wikipedia.org/wiki/Key_derivation_function) (KDFs). 

KDFs are cryptographic algorithms for deriving additional keys from a secret key and other data. A KDF allows keys to be created in a manner that is both secure and reproducible by two parties sharing knowledge of the inputs. Deriving keys is similar to hashing passwords. A KDF employs a keyed hash along with extra entropy from its other inputs to either derive new key material or safely expand existing values into a larger quantity of key material.

##### Operations

A key derivation function has two fundamental operations:

* *Instantiation and initialization*, creating KDF and initializing it with the appropriate parameters;
* *Derivation*, accepting key material and other optional inputs as well as parameters to describe the output, and generating the derived key or data.

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

We mentioned the advancements in quantum computing before in this article, and with good reason. Public-key based algorithms like Rivest-Shamir-Adleman (RSA) and Diffie-Hellman are more and more at risk of practical quantum computing attacks, but they are still in use by the Java Platform (for things like digitally signing JAR files or establishing secure network connections through TLS). To address this issue, [quantum-resistant](https://en.wikipedia.org/wiki/Post-quantum_cryptography) cryptographic algorithms have been invented, and this JEP introduces an implementation of one of those algorithms: the [Module-Lattice-Based Key-Encapsulation Mechanism](https://csrc.nist.gov/pubs/fips/203/final) (ML-KEM).

##### KEM Components

As described in [JEP 452](/2023/09/19/java-21-release-day/#jep-452-key-encapsulation-mechanism-api), any key encapsulation mechanism needs the following components:

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

You can use the ML-KEM KEM implementation to negotiate a shared secret key.

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

Java 23 contains a single feature that is part of the JLink tool:

* Linking Run-Time Images Without JMODs

#### JEP 493: Linking Run-Time Images Without JMODs

In cloud environments, container images that include an installed JDK are frequently copied over the network from container registries. 
Reducing the size of the JDK would improve the efficiency of these operations.
This is why the [`jlink`](https://dev.java/learn/jlink/) tool can now create custom run-time images without using the JDK's JMOD files.
This allows users to link a run-time image from modules, regardless of their source (be it JMOD files, modular JARs or part of a run-time image linked previously).

##### Redundancy

An installed JDK consists of a *run-time image* and a set of *packaged modules* in the JMOD format, for each module in the run-time image.
`jlink` uses the JMOD files when creating custom run-time images.
In fact, the run-time image in the JDK is itself a result from this process.
So every resource in the JDK's run-time image is also present in one of the JMOD files, which makes an installed JDK suffer from *redundancy*.
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
It will consume modules from the run-time image of which it is part only if the module `java.base` is not found on the module path. 
Any other modules must still be specified to `jlink` via the `--module-path` option.

##### Not enabled by default

This feature is currently not enabled by default, so the default JDKk build configuration will remain as it is today.
The resulting JDK will contain JMOD files and its `jlink` tool will not be able to operate without them. 
Whether the JDK build that you get from your preferred vendor contains this feature is up to that vendor.
The JEP states that the feature may however be enabled by default in a future release.

##### More Information

For more information on this feature, read [JEP 493](https://openjdk.org/jeps/493).

## Repreviews and finalizations

Now it's time to take a look at a few features that might already be familiar to you, because they were introduced in a previous version of Java. They have been repreviewed (or finalized) in Java 24, with only minor changes compared to Java 23 in most cases. Therefore, to avoid a very lengthy article, we'll outline these changes and link to a previous article for a full feature description, should you wish to refresh your memory.

### JEP 484: Class-File API

Java frameworks often use bytecode transformation to add functionality, relying on libraries like [ASM](https://asm.ow2.io/) or [Javassist](https://www.javassist.org/). However, the JDK's six-month release cycle can outpace these libraries, causing compatibility issues. JEP 484 addresses this by proposing a standard class-file API that evolves with the JDK, ensuring up-to-date class file processing.

#### What's Different From Java 23?

A few minor things changed to the API, based on feedback from the second preview stage. These changes mainly include the removal of a few fields, methods and interfaces, and the renaming of various enum values, fields and methods. The JEP includes [a detailed list](https://openjdk.org/jeps/484#Changes) on these changes.

#### More Information

For more information on this feature, read [JEP 484](https://openjdk.org/jeps/484) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-466-class-file-api-second-preview) from a previous article.

### JEP 485: Stream Gatherers

The Stream API offers a relatively diverse but predetermined range of intermediate operations, including mapping, filtering, sorting and more. JEP 485 introduces *stream gatherers*, which allow developers to define their own custom intermediate stream operations, so they can transform streams in their own preferred ways.

#### What's Different From Java 23?

Nothing was changed, apart from the fact that the API is now finalized in Java 24.

#### More Information

For more information on this feature, read [JEP 485](https://openjdk.org/jeps/485) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-473-stream-gatherers-second-preview) from a previous article.

### JEP 487: Scoped Values (Fourth Preview)

*Scoped values* enable the sharing of immutable data within and across threads. They are preferred to thread-local variables, especially when using large numbers of virtual threads.

#### What's Different From Java 23?

TODO

#### Preview Warning

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, read [JEP 487](https://openjdk.org/jeps/487) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-481-scoped-values-third-preview) from a previous article.

### JEP 488: Primitive Types in Patterns, instanceof and switch (Second Preview)

TODO

#### What's Different From Java 23?

TODO

#### Preview Warning

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, read [JEP 488](https://openjdk.org/jeps/488) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-455-primitive-types-in-patterns-instanceof-and-switch-preview) from a previous article.

### JEP 489: Vector API (Ninth Incubator)

TODO

#### What's Different From Java 23?

TODO

The Vector API will keep incubating until necessary features of Project Valhalla become available as preview features. When that happens, the Vector API will be adapted to use them, and it will be promoted from incubation to preview.

#### More Information

For more information on this feature, read [JEP 489](https://openjdk.org/jeps/489) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-469-vector-api-eighth-incubator) from a previous article.

### JEP 492: Flexible Constructor Bodies (Third Preview)

TODO

#### Preview Warning

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### What's Different From Java 23?

TODO

#### More Information

For more information on this feature, read [JEP 492](https://openjdk.org/jeps/492) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-482-flexible-constructor-bodies-second-preview) from a previous article.

### JEP 494: Module Import Declarations (Second Preview)

TODO

#### What's Different From Java 23?

TODO

#### More Information

For more information on this feature, read [JEP 494](https://openjdk.org/jeps/494) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-476-module-import-declarations-preview) from a previous article.

### JEP 495: Simple Source Files and Instance Main Methods (Fourth Preview)

TODO

#### What's Different From Java 23?

TODO

#### More Information

For more information on this feature, read [JEP 495](https://openjdk.org/jeps/495) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-477-implicitly-declared-classes-and-instance-main-methods-third-preview) from a previous article.

### JEP 499: Structured Concurrency (Fourth Preview)

TODO

#### What's Different From Java 23?

Nothing was changed, the API is re-previewed in Java 24 to give more time for feedback from real world usage.

TODO: add remark on any upcoming API changes

#### More Information

For more information on this feature, read [JEP 499](https://openjdk.org/jeps/499) or the [full feature description](/2024/09/17/java-23-has-arrived/#jep-480-structured-concurrency-third-preview) from a previous article.

## Deprecations & Restrictions

Java 24 also deprecates a few older features that weren't used that much and restricts a few other features that come with certain risks. Let's see which ones were involved in this effort to improve stability.

### JEP 472: Prepare to Restrict the Use of JNI

The Java Native Interface has been the factory standard of invoking foreign functions (outside of the JVM but on the same machine) for many years now. In Java 22 a more modern approach became available: the [Foreign Function & Memory API](/2024/03/19/java-22-is-here/#jep-454-foreign-function--memory-api). Although the new FFM API is the preferred alternative to JNI, that doesn't mean that JNI will be phased out. On the contrary, the Java language designers want to make sure that migrating from one to the other can be done easily. To make that happen, this JEP introduces certain warnings to JNI to mirror the warnings that the FFM API already produce. 

To be more precise, calling native code via JNI requires you to load a native library first and link a Java construct to a function in that library. JEP 472 will add the [restrictions](https://openjdk.org/jeps/454#Safety) that the FFM API already imposes on these loading and linking steps to JNI also. All warnings of this type aim to prepare developers for a future release that ensures [integrity by default](https://openjdk.org/jeps/8305968) by uniformly restricting JNI and the FFM API.

#### More Information

For more information on this feature, read [JEP 472](https://openjdk.org/jeps/472).

### JEP 479: Remove the Windows 32-bit x86 Port

This JEP removes the Windows 32-bit x86 port, which was to be expected after [its deprecation in Java 21](/2023/09/19/java-21-release-day/#jep-449-deprecate-the-windows-32-bit-x86-port-for-removal). This is due to the fact that Windows 10, the last Windows operating system to support 32-bit operation, [will reach end-of-life in October 2025](https://learn.microsoft.com/lifecycle/products/windows-10-home-and-pro). On top of that, the implementation of virtual threads on Windows 32-bit x86 is rudimentary to say the least: it uses a single platform thread for each virtual thread, effectively rendering the feature useless on this platform. So it's time to say goodbye to this port!

#### More Information

For more information on this removal, read [JEP 479](https://openjdk.org/jeps/479).

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
And so the memory-access methods in `sun.misc.Unsafe` became a valuable tool for library developers seeking greater power and performance than what standard APIs could provide.

Two standard APIs have emerged in recent years that are far better alternatives to these problematic methods:

* [`java.lang.invoke.VarHandle`](https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/lang/invoke/VarHandle.html), to manipulate on-heap memory safely and efficiently;
* [`java.lang.foreign.MemorySegment`](https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/lang/foreign/MemorySegment.html), to access off-heap memory safely and efficiently.

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

## Final thoughts

TODO - write outro that summarizes the article
TODO - scan article and group JEPs that make more sense to read in quick succession (like the security features KDF and the Quantum stuff)
TODO - replace all occurrences of `/202` with `https://hanno.codes/202`.