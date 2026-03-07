
---
layout: post
title: TODO
date: 17-03-2026 04:30:00 +0200
header:
  teaser: /assets/images/blog/TODO.jpg
excerpt: TODO
tags: 
- java
---

TODO: proofread the entire thing.

Java 26 ... TODO

![TODO](/assets/images/blog/todo.jpg)
> Photo by TODO, from <a href="https://www.pexels.com/photo/basketball-players-hugging-during-game-in-gym-5303477/">TODO</a>

## JEP Overview

To start off, let's look at an overview of the JEPs that ship with Java 26. This table contains the preview status for all JEP's, to which project they belong, what kind of features they add and the things that have changed since Java 25.

| JEP | Title                                                | Status             | Project       | Feature Type | Changes since previous Java version |
|-----|------------------------------------------------------|--------------------|---------------|--------------|-------------------------------------|
| **[500](#jep-500-prepare-to-make-final-mean-final)** | Prepare to Make Final Mean Final                     |                    | Core Libs     | Deprecation  | Warnings                            |
| **[504](#jep-504-remove-the-applet-api)** | Remove the Applet API                                |                    | Client Libs   | Deprecation  | Deprecation                         |
| **[516](#jep-516-ahead-of-time-object-caching-with-any-gc)** | Ahead-of-Time Object Caching with Any GC             |                    | HotSpot       | Performance  | New feature                         |
| **[517](#jep-517-http-3-for-the-http-client-api)** | HTTP 3 for the HTTP Client API                       |                    | Core Libs     | Extension    | New feature                         |
| **[522](#jep-522-g1-gc-improve-throughput-by-reducing-synchronization)** | G1 GC: Improve Throughput by Reducing Synchronization |                    | HotSpot       | Performance  | New feature                         |
| **[524](#jep-524-pem-encodings-of-cryptographic-objects)** | PEM Encodings of Cryptographic Objects               | Second Preview     | Security Libs | Security     | Minor                               |
| **[525](#jep-525-structured-concurrency)** | Structured Concurrency                               | Sixth Preview      | Loom          | Concurrency  | Minor                               |
| **[526](#jep-526-lazy-constants)** | Lazy Constants                                       | Second Preview     | Core Libs     | New API      | Major                               |
| **[529](#jep-529-vector-api)** | Vector API                                           | Eleventh Incubator | Panama        | New API      | None                                |
| **[530](#jep-530-primitive-types-in-patterns-instanceof-and-switch)** | Primitive Types in Patterns, instanceof, and switch  | Fourth Preview     | Amber         | Language     | Minor                               |

## New features

Let's start with the JEPs that add brand-new features to Java 26.

### HotSpot

Java 26 introduces two new features in [HotSpot](https://openjdk.org/groups/hotspot/):

* JEP 516: Ahead-of-Time Object Caching with Any GC
* JEP 522: G1 GC Improve Throughput by Reducing Synchronization

> The HotSpot JVM is the runtime engine that is developed by Oracle. It translates Java bytecode into machine code for the host operating system's processor architecture.

#### JEP 516: Ahead-of-Time Object Caching with Any GC

An important metric for applications that require a fast response time, such as web servers or real-time systems, is [tail latency](https://brooker.co.za/blog/2021/04/19/latency.html) (the time it takes for a request to be processed).
It can be caused by either garbage collection pauses, or requests that are sent to a new, not-yet-warmed-up JVM instance.
The first cause can be mitigated by using a low-latency garbage collector, such as ZGC, while the second can be mitigated by using the [ahead-of-time cache](https://hanno.codes/2025/03/18/java-24-rolls-out-today/#jep-483-ahead-of-time-class-loading--linking), which allows JVM instances to start up faster.

Java 24 introduced this ahead-of-time cache, storing classes in memory after reading, parsing, loading and linking them as a result of an initial *training run*.
Then, it could be re-used in subsequent runs of the application to improve startup time.
However, back then the use of the cache was somewhat limited, as cached Java objects were stored in a GC-specific format, making it incompatible with other garbage collectors like the Z Garbage Collector (ZGC).
JEP 516 extends support for the ahead-of-time cache to ZGC (and to any other garbage collector for that matter) by caching Java objects in a GC-agnostic format.

##### What Makes the New Cache Format GC-Agnostic?

Each garbage collector has its own policies for laying out objects in memory, which means that the memory addresses of cached objects are not valid across different garbage collectors.
To solve this problem, JEP 516 changes the cache format by replacing memory addresses with logical indices.
When the cache is loaded, these logical indices are converted back to memory addresses by *streaming* them into memory, materializing the cached objects in the process.

##### Usage

The JVM will automatically cache objects in the streamable, GC-agnostic format if, in training, either ZGC or the `-XX:-UseCompressedOops` command-line option was used, or if the heap was larger than 32GB. 
In contrast, it will cache objects in the old, GC-specific format if, in training, the `-XX:+UseCompressedOops` (notice the ' plus') command-line option was used. This indicates that the training environment had a heap smaller than 32GB and did not use ZGC. 

If you want to force the use of the new GC-agnostic cache format regardless of the training environment, you can specify the `-XX:+AOTStreamableObjects` command-line option to make it happen.

##### Why Not Simply Create ZGC-Specific Caches?

The alternative of creating ZGC-specific caches was not pursued because it would have required maintaining separate caches for each garbage collector. Moreover, the only benefit of a ZGC-specific cache would have been a slightly better performance on single-core machines. This benefit would have been negligible in practice, as the highly-concurrent ZGC was designed to perform well on multi-core machines rather than single-core machines, and it would not have justified the maintenance cost of multiple caches.

##### More Information

For more information on this feature, read [JEP 516](https://openjdk.org/jeps/516).

#### JEP 522: G1 GC: Improve Throughput by Reducing Synchronization

*G1 GC* has been Java's [default garbage collector since Java 9](https://openjdk.org/jeps/248). It's been designed to provide high performance and low pause times for applications with large heaps, with the aim of balancing latency and throughput. To achieve this balance, G1 performs its work concurrently with the application, making the application threads share the CPU with GC threads. This situation requires thread synchronization, which unfortunately lowers throughput and increases latency.

JEP 522 proposes to improve both throughput and latency by reducing the amount of synchronization required between application threads and GC threads.

##### Why Is Synchronization Currently Necessary?

When G1 reclaims memory, live objects in the heap are copied to new memory regions, freeing up the space they leave behind. References to those objects must be updated to point to their new location. To prevent having to scan the entire heap for existing references to these objects, G1 maintains a data structure called the *card table*, which is updated every time an object reference is stored in a field. These updates are performed by pieces of code called *write barriers*, which G1 injects into the application in cooperation with the JIT.

Scanning the card table is an efficient operation and will typically fit within a GC pause's time window. However, in  environments where objects are allocated very frequently the card table may grow too large to be scanned within the timespan of G1's pause time goal. To avoid that, G1 optimizes the card table in the background via separate optimizer threads. This approach can only work if the card table is updated in a thread-safe way, which is currently achieved by synchronizing the optimizer threads with the application threads. Arguably this can lead to more complicated and slower write-barrier code.

##### Towards A Second Card Table

JEP 522 proposes to introduce a *second card table*, to make sure that the optimizer threads and applications threads no longer interfere. The write barriers in the application threads will update the first card table without any synchronization, while the optimizer threads will update the second, initially empty, card table.

When G1 determines that scanning the current card table during a pause would likely breach the pause‑time target, it atomically switches the two card tables. Application threads then continue to write to the now‑empty table (the former “second” table), while dedicated optimizer threads process the previously‑filled table (the former “first” table) without any additional synchronization. G1 repeats this swapping as needed so that the work required on the active card table stays within the desired limits.

This approach reduces the amount of synchronization required between application and optimizer threads. In applications that heavily modify object-reference fields, throughput gains of 5-15% can be expected. On top of that, because the write barrier code can be a lot simpler, additional throughput gains of up to 5% have been observed in x64 architectures, even in applications that don't heavily modify object-reference fields.

The two card tables are identical in size, each consuming the same extra native memory. Together they occupy roughly 0.2% of the Java heap, which translates to about 2MB of native memory for every gigabyte of heap space. This modest overhead is well worth the sizable performance gains—especially when you consider that, before Java 20, G1 needed more than eight times the memory that the second card table now requires.

##### More Information

For more information on this feature, read [JEP 522](https://openjdk.org/jeps/522).

### Core Libs

Java 26 introduces a single new feature that is part of the Core Libs:

* JEP 517: HTTP 3 for the HTTP Client API

#### JEP 517: HTTP 3 for the HTTP Client API

TODO

##### More Information

For more information on this feature, read [JEP 517](https://openjdk.org/jeps/517).

## Repreviews

Now it's time to take a look at a few features that might already be familiar to you, because they were introduced in a previous version of Java. They have been repreviewed in Java 26, with only minor changes compared to Java 25 in most cases. Therefore, to avoid a very lengthy article, we'll outline these changes and link to a previous article for a full feature description, should you wish to refresh your memory.

### JEP 524: PEM Encodings of Cryptographic Objects (Second Preview)

Within a Java context, cryptographic objects such as public keys, private keys and certificates can be easily created and distributed. But outside of the Java world, the de facto standard is the [Privacy-Enhanced Mail](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) (PEM) format. Let's see an example of a PEM-encoded cryptographic object:

```
-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEi/kRGOL7wCPTN4KJ2ppeSt5UYB6u
cPjjuKDtFTXbguOIFDdZ65O/8HTUqS/sVzRF+dg7H3/tkQ/36KdtuADbwQ==
-----END PUBLIC KEY-----
```

The Java Platform currently doesn't include an easy-to-use API for decoding and encoding text in the PEM format, which means that decoding a PEM-encoded key can be a tedious job that involves careful parsing of the source PEM text. To further illustrate this point, encrypting and decrypting a private key currently requires over a dozen lines of code.

To solve this problem, JEP 524 introduces an API that can encode objects to the PEM format. It effectively acts as a bridge between Base64 and cryptographic objects. It involves a new interface and three new classes, in the `java.security` package:

[`DEREncodable`](https://cr.openjdk.org/~ascarpino/pem26/api/java.base/java/security/DEREncodable.html)
: A sealed interface that groups together all cryptographic objects that support converting their instances to and from byte arrays in the [Distinguished Encoding Rules](https://en.wikipedia.org/wiki/X.690#DER_encoding) (DER) format.

[`PEMEncoder`](https://cr.openjdk.org/~ascarpino/pem26/api/java.base/java/security/PEMEncoder.html)
: A class that declares methods for encoding `DEREncodable` objects into PEM text.

[`PEMDecoder`](https://cr.openjdk.org/~ascarpino/pem26/api/java.base/java/security/PEMDecoder.html)
: A class that declares methods for decoding PEM text to `DEREncodable` objects.

[`PEM`](https://cr.openjdk.org/~ascarpino/pem26/api/java.base/java/security/PEM.html)
: A record that implements `DEREncodable`, which can hold any type of PEM data. It allows you to encode and decode PEM tests yielding cryptographic objects for which no Java representation currently exists. 

##### Typical Usage

The following code example shows typical usage of the API:

```java
PrivateKey privateKey = ...;
PublicKey publicKey = ...;

// let's encode a cryptographic object!
PEMEncoder pemEncoder = PEMEncoder.of();

// this returns PEM text in a byte array
byte[] privateKeyPem = pemEncoder.encode(privateKey);

// this returns PEM text in a String
String keyPairPem = pemEncoder.encodeToString(new KeyPair(privateKey, publicKey)); 

// this returns encrypted PEM text
String password = "java-first-java-always";
String pem = pemEncoder.withEncryption(password).encodeToString(privateKey);

// let's decode a cryptographic object!
PEMDecoder pemDecoder = PEMDecoder.of();

// this returns a DEREncodable, so we need to pattern-match
switch (pemDecoder.decode(pem)) {
    case PublicKey publicKey -> ...;
    case PrivateKey privateKey -> ...;
    default -> throw new IllegalArgumentException("Unsupported cryptographic object");
}

// alternatively, if you know the type of the encoded cryptographic object in advance:
PrivateKey key = pemDecoder.decode(pem, PrivateKey.class);

// this decodes an encrypted cryptographic object
PrivateKey decryptedkey = pemDecoder.withDecryption(password).decode(pem, PrivateKey.class);
```

##### Preview Warning

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### What's Different From Java 25?

A few minor changes were made to the API compared to Java 25:

* `PEMRecord` was renamed to `PEM`, and includes a decode() method that returns decoded Base64 content;
* The `PEMEncoder` and `PEMDecoder` classes now support the encryption and decryption of `KeyPair` and `PKCS8EncodedKeySpec` objects;
* Finally, a few changes were made to the `EncryptedPrivateKeyInfo` class:
  * The `encryptKey` methods are now named `encrypt`, and they now accept `DEREncodable` objects rather than `PrivateKey` objects, enabling the encryption of `KeyPair` and `PKCS8EncodedKeySpec` objects.
  * It includes `getKeyPair` methods that decrypt PKCS#8-encoded text containing a `PublicKey`.
  * The exceptions thrown by the `getKey` methods are now aligned with those thrown by the nearby `getKeySpec` methods.

#### More Information

For more information on this feature, see [JEP 524](https://openjdk.org/jeps/524).

### JEP 525: Structured Concurrency (Sixth Preview)

Java's take on concurrency has always been _unstructured_, meaning that tasks run independently of each other. There's no hierarchy, scope, or other structure involved, which means errors or cancellation intent is hard to communicate. To illustrate this, let's look at a code example that takes place in a restaurant:

> These code examples were taken from my conference talk ["Java's Concurrency Journey Continues! Exploring Structured Concurrency and Scoped Values"](https://hanno.codes/talks/#javas-concurrency-journey-continues).

```java
public class MultiWaiterRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            Future<Course> starter = executor.submit(() -> grover.announceCourse(CourseType.STARTER));
            Future<Course> main = executor.submit(() -> zoe.announceCourse(CourseType.MAIN));
            Future<Course> dessert = executor.submit(() -> rosita.announceCourse(CourseType.DESSERT));

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
        }
    }
}
```

Note that the `announceCourse(..)` method in the `Waiter` class sometimes fails with an `OutOfStockException`, because one of the ingredients for the course might not be in stock. This can lead to some problems:

* If `zoe.announceCourse(CourseType.MAIN)` takes a long time to execute but `grover.announceCourse(CourseType.STARTER)` fails in the meantime, the `announceMenu(..)` method will unnecessarily wait for the main course announcement by blocking on `main.get()`, instead of cancelling it (which would be the sensible thing to do).
* If an exception happens in `zoe.announceCourse(CourseType.MAIN)`, `main.get()` will throw it, but `grover.announceCourse(CourseType.STARTER)` will continue to run in its own thread, resulting in thread leakage.
* If the thread executing `announceMenu(..)` is interrupted, the interruption will not propagate to the subtasks: all threads that run an `announceCourse(..)` invocation will leak, continuing to run even after `announceMenu()` has failed.

Ultimately the problem here is that our program is logically structured with task-subtask relationships, but these relationships exist only in the mind of the developer. We might all prefer structured code that reads like a sequential story, but this example simply doesn't meet that criterion.

In contrast, the execution of single-threaded code _always_ enforces a hierarchy of tasks and subtasks, as shown by the single-threaded version of our restaurant example:

```java
public class SingleWaiterRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws OutOfStockException {
        Waiter elmo = new Waiter("Elmo");

        Course starter = elmo.announceCourse(CourseType.STARTER);
        Course main = elmo.announceCourse(CourseType.MAIN);
        Course dessert = elmo.announceCourse(CourseType.DESSERT);

        return new MultiCourseMeal(starter, main, dessert);
    }
}
```

Here, we don't have _any_ of the problems we had before.
Our waiter Elmo will announce the courses in exactly the right order, and if one subtask fails the remaining one(s) won't even be started.
And because all work runs in the same thread, there is no risk of thread leakage.

It became apparent from these examples that concurrent programming would be a lot easier and more intuitive if enforcing the hierarchy of tasks and subtasks was possible, just like with single-threaded code.

#### Introducing Structured Concurrency

In a structured concurrency approach, threads have a clear hierarchy, their own scope, and clear entry and exit points. Structured concurrency arranges threads hierarchically, akin to function calls, forming a tree with parent-child relationships. Execution scopes persist until all child threads complete, matching code structure.

#### Shutdown on Failure

Let's look at a structured, concurrent version of our example now:

```java
public class StructuredConcurrencyRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var scope = StructuredTaskScope.open()) {
            Supplier<Course> starter = scope.fork(() -> grover.announceCourse(CourseType.STARTER));
            Supplier<Course> main = scope.fork(() -> zoe.announceCourse(CourseType.MAIN));
            Supplier<Course> dessert = scope.fork(() -> rosita.announceCourse(CourseType.DESSERT));

            scope.join(); // 1

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get()); // 2
        }
    }
}
```

The scope’s purpose is to keep the threads together. At `1`, we wait (`join`) until all threads are done with their work. If one of the threads is interrupted, an `InterruptedException` is thrown. A `RuntimeException` can also be thrown here, if an exception occurs in one of the spawned threads. Once we reach `2`, we can be sure everything has gone well, and we can retrieve and process the results.

Actually, the main difference with the code we had before is the fact that we create threads (`fork`) within a new `scope`. Now we can be certain that the lifetimes of the three threads are confined to this scope, which coincides with the body of the try-with-resources statement.

Furthermore, we've gained _short-circuiting behaviour_. When one of the `announceCourse(..)` subtasks fails, the others are canceled if they didn't complete yet. We've also gained _cancellation propagation_. When the thread that runs `announceMenu()` is interrupted before or during the call to `scope.join()`, all subtasks are cancelled automatically when the thread exits the scope.

#### Shutdown on Success

The factory method that gave us the scope (`StructuredTaskScope.open()`) implements a shutdown-on-failure policy by default, which cancels any remaining tasks in the scope if one of the tasks has failed. A shutdown-on-success policy is also available: it cancels any remaining tasks in the scope if one of the tasks has succeeded. It can be used to avoid doing unnecessary work when a successful result has already been achieved. Which would actually be a perfect way to solve the problems that our patient waiter from the article introduction was experiencing!

We can use a shutdown-on-success policy by calling an overload of the `StructuredTaskScope.open()` method that takes a `Joiner` as its parameter. Let’s see what that would look like:

```java
record DrinkOrder(Guest guest, Drink drink) {}

public class StructuredConcurrencyBar implements Bar {
    @Override
    public DrinkOrder determineDrinkOrder(Guest guest) throws InterruptedException, ExecutionException {
        Waiter zoe = new Waiter("Zoe");
        Waiter elmo = new Waiter("Elmo");

        try (var scope = StructuredTaskScope.open(Joiner.<DrinkOrder>anySuccessfulOrThrow())) {
            scope.fork(() -> zoe.getDrinkOrder(guest, BEER, WINE, JUICE));
            scope.fork(() -> elmo.getDrinkOrder(guest, COFFEE, TEA, COCKTAIL, DISTILLED));

            return scope.join(); // 1
        }
    }
}
```

In this example the waiter is responsible for getting a valid `DrinkOrder` object based on guest preference and the drinks supply at the bar.
In the method `Waiter.getDrinkOrder(Guest guest, DrinkCategory... categories)`, the waiter starts to list all available drinks in the drink categories that were passed to the method.
Once a guest hears something they like, they respond and the waiter creates a drink order. When this happens, the `getDrinkOrder(..)` method returns a `DrinkOrder` object and the scope will shut down. 
This means that any unfinished subtasks (such as the one in which Elmo is still listing different kinds of tea) will be cancelled.
The `join()` method at `1` will either return a valid `DrinkOrder` object, or throw a `RuntimeException` if one of the subtasks has failed.

#### More Shutdown Policies

We’ve seen examples of two shutdown policies so far, but four more are provided out-of-the-box through the static factory methods in the `StructuredTaskScope.Joiner` interface. For example, `Joiner.allSuccessfulOrThrow()` will keep the scope alive until all subtasks have completed successfully, and cancels it if any subtasks fails. And `Joiner.awaitAll()` will wait for all subtasks to complete, whether they complete successfully or not. It’s also possible to create your own shutdown policies by implementing the `Joiner` interface. That will allow you to have full control over when the scope will be shut down and what results will be collected.

#### What's Different From Java 25?

A few minor changes were made to the API compared to Java 25:

* A new method in the `Joiner` interface, `onTimeout()`, allows implementations of that interface to return a result when a timeout expires.
* `Joiner::allSuccessfulOrThrow()` now returns a list of results instead of a stream of subtasks.
* `Joiner::anySuccessfulResultOrThrow()` was renamed to the slightly simpler `anySuccessfulOrThrow()`.
* The static `open` method that used to take a `Joiner` and a `Function` to modify the default configuration now takes a `Joiner` and a  `UnaryOperator`.

#### Preview Warning

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

[JEP 525](https://openjdk.org/jeps/525) has more details on the current state of this feature, should you wish to learn more.

### JEP 526: Lazy Constants (Second Preview)

Immutable objects are a far less complicated concept than mutable objects, because they can only be in a single state and can be shared freely across multiple threads.
Currently, the main tool to achieve immutability in Java is `final` fields.
But they come with two drawbacks, restricting their potential in many real-world applications:

* they must be set eagerly;
* the order in which multiple `final` fields are initialized can never be changed, as it is determined by the [textual order](https://docs.oracle.com/javase/specs/jls/se23/html/jls-12.html#jls-12.4) in which the fields are declared.

Consider the use of immutability in the following code example, which takes place in a guitar store domain:

```java
class OrderController {
    private final Logger logger = Logger.create(OrderController.class);

    void submitOrder(User user, List<Guitar> guitar) {
        logger.info("Ordering new guitars...");
        
        // ...
        
        logger.info("New guitars have been ordered, let's get to work!");
    }
}
```

Whenever an instance of `OrderController` is created, the `logger` field is initialized eagerly, which potentially makes creating an `OrderController` slow.
And this might not be the only place in our application where a `logger` field is initialized eagerly:

```java
class GuitarStore {
    static final OrderController ORDERS = new OrderController();
    static final GuitarRepository GUITARS = new GuitarRepository();
    static final ManufacturerService MANUFACTURERS = new ManufacturerService();
}
```

All this initialization work causes the application to start up more slowly, and the worst thing is: it may not even be necessary!
If a user is simply browsing the guitar store, with no intention of ordering a new guitar, the `OrderController` won't even be called and we will have initialized the `logger` field for nothing.

##### Sacrificing Immutability For More Flexible Initialization

The only alternative we currently have is to resort to a mutability-based approach, in which we delay the initialization of complex objects to as late a time as possible:

```java
class OrderController {
    private Logger logger;

    Logger getLogger() {
        if (logger == null) {
            logger = Logger.create(OrderController.class);
        }
        return logger;
    }

    void submitOrder(User user, List<Guitar> guitar) {
        getLogger().info("Ordering new guitars...");
        
        // ...
        
        getLogger().info("New guitars have been ordered, let's get to work!");
    }
}
```

This improves application startup, but comes with a few drawbacks of its own:

* All accesses to the `logger` field must go through the `getLogger` method, but code that fails to follow this practice runs the risk of encountering `NullPointerException`s;
* In multi-threaded environments, multiple logger objects could be created during concurrent calls to the `submitOrder` method;
* [Constant-folding](https://en.wikipedia.org/wiki/Constant_folding) access to an already-initialized `logger` field is no longer viable, as the JVM can't trust its content never to change after its initial update.

What we need is a solution that has the best of both worlds: 

* a way to promise that a field will be initialized by the time it is used,
* with a value that is computed at most once, and
* safely with respect to concurrency.

In other words, we want to *defer immutability*, and have first-class support for it in the Java runtime.

##### Lazy Constants

JEP 526 introduces that first-class support in the form of *lazy constants*.
A lazy constant is an object of type `LazyConstant`, that holds a single data value.
It must be initialized some time before its content is first retrieved, and it is immutable thereafter.

Let's rewrite the `OrderController` class to use a lazy constant for its logger:

```java
class OrderController {
    private final LazyConstant<Logger> logger = LazyConstant.of(() -> Logger.create(OrderController.class));

    void submitOrder(User user, List<Guitar> guitar) {
        logger.get().info("Ordering new guitars...");
        
        // ...
        
        logger.get().info("New guitars have been ordered, let's get to work!");
    }
}
```

Initially, the lazy constant is uninitialized. When it is accessed for the first time through the `get()` method, it is initialized by invoking the lambda expression that was passed to the `of()` factory method.
If the lazy constant was already initialized, then the `get` method simply returns its content.
Thus, the `get` method guarantees that the provided lambda expression is evaluated only once (even when it is invoked concurrently).

If we look at the properties of lazy constants, we see that they fill a gap between final and non-final fields:

|  | **Update count** | **Update location** | **Constant folding?** | **Concurrent updates?** |
|-:|-----------------:|--------------------:|----------------------:|------------------------:|
| `final` field | 1 | Constructor or static initializer | Yes | No |
| `LazyConstant` | [0, 1] | Computing function | Yes, after update | Yes, by winner |
| non-`final` field | [0, ∞] | Anywhere | No | Yes |

Usage of lazy constants is certainly not limited to loggers–we can also use a lazy constant to store the `OrderController` component itself, and related components:

```java
class GuitarStore {
    static final LazyConstant<OrderController> ORDERS = LazyConstant.of(OrderController::new);
    static final LazyConstant<GuitarRepository> GUITARS = LazyConstant.of(GuitarRepository::new);
    static final LazyConstant<ManufacturerService> MANUFACTURERS = LazyConstant.of(ManufacturerService::new);

    public static OrderController orders() {
        return ORDERS.get();
    }

    public static GuitarRepository guitars() {
        return GUITARS.get();
    }

    public static ManufacturerService manufacturers() {
        return MANUFACTURERS.get();
    }
}
```

The application's startup time improves because it no longer initializes its components, such as `OrderController`, up front. 
Rather, it initializes each component on demand, via the `get` method of the corresponding lazy constant. 
Each component, moreover, initializes its sub-components, such as its logger, on demand in the same way.

Under the hood, the JVM will treat the content of any lazy constant that is declared as `final` as a constant, allowing constant-folding optimizations to happen.

##### Lazy Lists

What if you wanted to keep track of multiple lazy constants, for example when keeping a pool of objects?
We can achieve this by using a *lazy list*:

```java
class GuitarStore {
    static final int POOL_SIZE = 10;
    static final List<OrderController> ORDERS = List.ofLazy(POOL_SIZE, _ -> new OrderController());

    public static OrderController orders() {
        long index = Thread.currentThread().threadId() % POOL_SIZE;
        return ORDERS.get((int) index);
    }
}
```

Here, `ORDERS` is no longer a lazy constant, but a lazy list, in which each element is stored in a lazy constant.
To access the content, clients call `ORDERS.get(...)`, passing it an index, of which the first invocation will invoke the lamdba function that ignores the index and invokes the `OrderController()` constructor.
Subsequent invocations of `ORDERS.get(...)` with the same index will return the element's content immediately.

##### Lazy Maps

Alternatively, we could have solved the problem with a _lazy map_, whose keys are known at construction time and whose values are stored in lazy constants, initialized on demand by a computing function that is also provided at construction:

```java
class GuitarStore {
    static final Map<String, OrderController> ORDERS = Map.ofLazy(Set.of("Customers", "Internal", "Testing"), _ -> new OrderController());

    public static OrderController orders() {
        return ORDERS.get(Thread.currentThread().getName());
    }
}
```

In this example, `OrderController` instances are associated with thread names ("Customers", "Internal", and "Testing" in this case) rather than integer indexes computed from thread identifiers. Lazy maps allow for more expressive access idioms than lazy lists, but otherwise have all the same benefits.

#### What's Different From Java 25?

The feature that used to be known as 'stable values' was renamed to 'lazy constants' to better capture its intended high-level use case.

Other changes have a similar purpose–they include:
* Removing the low-level methods `orElseSet`, `setOrThrow`, and `trySet`, leaving only factory methods that take value-computing functions;
* Moving the factory methods for lazy lists (`StableValue.list`) and maps (`StableValue.map`) into the `List` and `Map` interfaces, respectively, to enhance discoverability;
* Incorporating the ideas behind 'stable suppliers' into the new `LazyConstant.get()` method;
* Removing the `function` and `intFunction` factory methods to further simplify the API;
* Disallowing `null` as a computed value in order to improve performance and better align lazy constants with constructs such as unmodifiable collections and `ScopedValue`s.

#### More Information

[JEP 526](https://openjdk.org/jeps/526) has more details on the current state of this feature, should you wish to learn more.

### JEP 529: Vector API (Eleventh Incubator)

The Vector API makes it possible to express vector computations that reliably compile at runtime to optimal vector instructions. 
This means that these computations will significantly outperform equivalent scalar computations on the supported CPU architectures (x64 and AArch64).

#### What's Different From Java 25?

The following changes were made to the Vector API compared to Java 24:

TODO

The Vector API will keep incubating until necessary features of Project Valhalla become available as preview features. When that happens, the Vector API will be adapted to use them, and it will be promoted from incubation to preview.

#### More Information

For more information on this feature, read [JEP 529](https://openjdk.org/jeps/529) or the [full feature description](https://hanno.codes/2024/09/17/java-23-has-arrived/#jep-469-vector-api-eighth-incubator) from a previous article.

### JEP 530: Primitive Types in Patterns, instanceof, and switch (Fourth Preview)

Pattern matching now supports primitive types in all pattern contexts. On top of that, the `instanceof` and `switch` constructs have been extended to also work with all primitive types.

TODO: potentially include the full feature description here (from a previous article) with updated code examples. In that case: remove the link to the previous article in the "More Information" section.

#### What's Different From Java 25?

TODO

#### Preview Warning

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, read [JEP 530](https://openjdk.org/jeps/530) or the [full feature description](https://hanno.codes/2024/09/17/java-23-has-arrived/#jep-455-primitive-types-in-patterns-instanceof-and-switch-preview) from a previous article.

## Deprecations

Java 26 also deprecates a few older features that weren't used that much. Let's see which ones were involved in this effort to improve stability.

### JEP 500: Prepare to Make Final Mean Final

TODO: paraphrase
- future versions of Java will disallow the mutation of final fields by deep reflection.
- final fields are important when reasoning about correctness and for performance reasions (more optimizations possible, like constant folding)
- Unfortunately, the expectation that a final field cannot be reassigned is false. Several APIs allow final fields to be reassigned at any time by any code in a program, undermining all reasoning about correctness and invalidating important optimizations. 
- The deep reflection API is the most notorious, embodied in the `Field.setAccessible` and `Field.set` methods.
- <code example>
- The possibility was added so that serialization libraries could mutate final fields when initializing objects during deserialization.
- In retrospect, offering such unconstrained functionality was a poor choice because it sacrificed integrity.
- Mutating final fields is currently not allowed in hidden classes and in records, and now it's time to extend this behavior to regular classes.

#### More Information

For more information on this removal, read [JEP 500](https://openjdk.org/jeps/500).

### JEP 504: Remove the Applet API

TODO

#### More Information

For more information on this removal, read [JEP 504](https://openjdk.org/jeps/504).

## Final thoughts

TODO
