---
layout: post
title: TODO
date: 15-09-2026 04:30:00 +0200
header:
  teaser: /assets/images/blog/todo.jpg
excerpt: TODO
tags: 
- java
---

Java 27 ... TODO

![TODO](/assets/images/blog/todo.jpg)
> Photo by TODO, from <a href="https://www.pexels.com/photo/basketball-players-hugging-during-game-in-gym-5303477/">TODO</a>

TODO: full read-through

## JEP Overview

To start off, let's look at an overview of the JEPs that ship with Java 27. This table contains the preview status for all JEPs, to which project they belong, what kind of features they add and the things that have changed since Java 26.

TODO

## New Features

Let's start with the JEPs that add brand-new features to Java 27.

### JEP 527: Post-Quantum Hybrid Key Exchange for TLS 1.3

TODO

##### More Information

For more information on this feature, read [JEP TODO](https://openjdk.org/jeps/todo).

### JEP 536: JFR In-Process Data Redaction

TODO

##### More Information

For more information on this feature, read [JEP TODO](https://openjdk.org/jeps/todo).

## Repreviews

Now it's time to take a look at a few features that might already be familiar to you, because they were introduced in a previous version of Java. They have been repreviewed in Java 27, with only minor changes compared to Java 26 in most cases.

### JEP 531: Lazy Constants (Third Preview)


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

* a way to promise that a field will be initialized by the time it is used;
* with a value that is computed at most once, and;
* safely with respect to concurrency.

In other words, we want to *defer immutability*, and have first-class support for it in the Java runtime.

##### Lazy Constants

JEP 526 introduces that first-class support in the form of *lazy constants*.
A lazy constant is an object of type `LazyConstant`, that holds a single data value.
It must be initialized some time before its content is first retrieved, and is immutable thereafter.

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

|                   | **Update count** |               **Update location** | **Constant folding?** | **Concurrent updates?** |
| ----------------: | ---------------: | --------------------------------: | --------------------: | ----------------------: |
|     `final` field |                1 | Constructor or static initializer |                   Yes |                      No |
|    `LazyConstant` |           [0, 1] |                Computing function |     Yes, after update |          Yes, by winner |
| non-`final` field |           [0, ∞] |                          Anywhere |                    No |                     Yes |

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

Under the hood, the JVM treats the content of any lazy constant that is declared as `final` as a constant, allowing constant-folding optimizations to happen.

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
To access the content, clients call `ORDERS.get(...)`, passing it an index, of which the first invocation will invoke the lambda function that ignores the index and invokes the `OrderController()` constructor.
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

##### Lazy Sets

TODO

#### What's Different From Java 26?

The API [was changed significantly in Java 26](TODO), shifting the feature's focus to high-level use cases only. 
The minor changes applied in Java 27 have a similar purpose–they include:

* Removing the low-level methods `isInitialized` and `orElse`, as these could be used in ways not consistent with the design goals of the API.
* Adding a new factory method, `Set.ofLazy(...)`, that can create a stable `Set` of pre-defined element candidates. With this addition, there lazy versions of the three fundamental collection types now exist: `List`, `Set`, and `Map`.

#### More Information

[JEP 531](https://openjdk.org/jeps/531) has more details on the current state of this feature, should you wish to learn more.

### JEP 532: Primitive Types in Patterns, instanceof, and switch (Fifth Preview)

Since Java 23, pattern matching supports primitive types in all pattern contexts, and in the `instanceof` and `switch` constructs. The feature has been in three consecutive preview statuses, and will be previewed for a fourth time in Java 26. Let's first go through the differences with Java 22 before we highlight the changes in the fourth preview.

#### Pattern Matching for Switch

Java 22's version of [pattern matching for switch](https://openjdk.org/jeps/441) didn't support type patterns that specify a primitive type. In Java 23 support was added for primitive type patterns in `switch`, allowing the following code example:

```java
switch (reverb.roomSize()) {
    case 1 -> "Toilet";
    case 2 -> "Bedroom";
    case 30 -> "Classroom";
    default -> "Unsupported value: " + reverb.roomSize();
}
```

...to be written as follows:

```java
switch (reverb.roomSize()) {
    case 1 -> "Toilet";
    case 2 -> "Bedroom";
    case 30 -> "Classroom";
    case int i -> "Unsupported int value: " + i;
}
```

This also allows guards to inspect the matched value, like so:

```java
switch (reverb.roomSize()) {
    case 1 -> "Toilet";
    case 2 -> "Bedroom";
    case 30 -> "Classroom";
    case int i when i > 100 && i < 1000 -> "Cinema";
    case int i when i > 5000 -> "Stadium";
    case int i -> "Unsupported int value: " + i;
}
```

#### Record Patterns

[Record patterns](https://openjdk.org/jeps/440) used to have limited support for primitive types.
Recall that a record pattern decomposes a record into its individual components, but when one of them is a primitive type, the record pattern must be precise about its type. To illustrate this point, consider the following code example:

```java
record Tuner(double pitchInHz) implements Effect {}

var tuner = new Tuner(440); // int argument is widened to double

// Attempt 1: record pattern match on int argument
if (tuner instanceof Tuner(int p)) {} // doesn't compile!

// Attempt 2: record pattern match on double argument
if (tuner instanceof Tuner(double p)) {
    int pitch = p; // doesn't compile! needs a cast to int
}

// Attempt 3: record pattern match on double argument, cast to int
if (tuner instanceof Tuner(double p)) {
    int pitch = (int) p;
}
```

To put it differently, the Java compiler widens the provided `int` to a `double`, but it doesn't narrow it back to an `int`. This limitation exists because narrowing could lead to data loss: the value of the `double` at runtime might exceed the range of an `int` or have more precision than an `int` can accommodate. However, one significant advantage of pattern matching is its ability to automatically reject invalid values by not matching them at all. If the `double` component of a `Tuner` is either too large or too precise to safely convert back to an `int`, then `instanceof Tuner(int p)` would simply return `false`, allowing the program to manage the large `double` component in a different code branch.

This is analogous to how pattern matching currently behaves for reference type patterns. For example:

```java
record SingleEffect(Effect effect) {}
var singleEffect = new SingleEffect(...);

if (singleEffect instanceof SingleEffect(Delay d)) {
    // ...
} else if (singleEffect instanceof SingleEffect(Reverb r)) {
    // ...
} else {
    // ...
}
```
`instanceof` can be used here to try to match a `SingleEffect` with a `Delay` or a `Reverb` component; it automatically narrows if the pattern matches.

To summarize, the JEP proposes to make primitive type patterns work as smoothly as reference type patterns, allowing `Tuner(int p)` even if the corresponding record component is a numeric primitive type other than `int`.

#### Pattern Matching for instanceof

The Java 22-version of [pattern matching for instanceof](https://openjdk.org/jeps/394) didn't support primitive type patterns, but this capability would perfectly align with the purpose of `instanceof`: to test whether a value can be converted safely to a given type. To convert primitives safely, Java developers had to deal with lossy casts and range checks to prevent loss of information:

```java
int roomSize = reverb.roomSize();

if (roomSize >= -128 && roomSize < 127) {
    byte r = (byte) roomSize;
    // now it's safe to use r
}
```

The JEP proposes the possibility to replace these constructs with simple `instanceof` checks that operate on primitives. Let's rewrite the code example to make use of this feature:

```java
int roomSize = reverb.roomSize();

if (roomSize instanceof byte r) {
    // now it's safe to use r
}
```

The pattern `roomSize instanceof byte r` will match only if `roomSize` fits into a `byte`, eliminating the need for casts and range checks.

#### Primitive Types in instanceof

The `instanceof` keyword used to take a reference type only, and since Java 16 it can also take a type pattern.
But it would make sense to have `instanceof` take a primitive type also.
In that case `instanceof` would check if the conversion is safe but would not actually perform it:

```java
if (roomSize instanceof byte) { // check if value of roomSize fits in a byte
    ... (byte) roomSize ... // yes, it fits! but cast is required
}
```

The JEP proposes to support this construct, which makes it easier to change the `instanceof` check to take a type pattern and vice versa.

#### Primitive Types in switch

The Java 22-version of the `switch` statement/expression supported `byte`, `short`, `char`, and `int` values.
The JEP proposes to also add support for the other primitive types: `boolean`, `float`, `double` and `long`.
A `switch` on a `boolean` value can be a good alternative for the ternary operator (`?:`), because its branches can also hold statements instead of just expressions.

```java
String guitaristResponse = switch (guitar.isInTune()) {
    case true -> "Ready to play a song.";
    case false -> {
        log.warn("Guitar is out of tune!");
        yield "Let's take five!";
    }
}
```

#### What's Different From Java 26?

Compared to the fourth preview version of this feature in Java 26, nothing was changed or added.

#### Preview Warning

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, read [JEP 532](https://openjdk.org/jeps/532).

### JEP 533: Structured Concurrency (Seventh Preview)

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

The scope's purpose is to keep the threads together. At `1`, we wait (`join`) until all threads are done with their work. If one of the threads is interrupted, an `InterruptedException` is thrown. A `RuntimeException` can also be thrown here, if an exception occurs in one of the spawned threads. Once we reach `2`, we can be sure everything has gone well, and we can retrieve and process the results.

Actually, the main difference with the code we had before is the fact that we create threads (`fork`) within a new `scope`. Now we can be certain that the lifetimes of the three threads are confined to this scope, which coincides with the body of the try-with-resources statement.

Furthermore, we've gained _short-circuiting behaviour_. When one of the `announceCourse(..)` subtasks fails, the others are canceled if they didn't complete yet. We've also gained _cancellation propagation_. When the thread that runs `announceMenu()` is interrupted before or during the call to `scope.join()`, all subtasks are cancelled automatically when the thread exits the scope.

#### Shutdown on Success

The factory method that gave us the scope (`StructuredTaskScope.open()`) implements a shutdown-on-failure policy by default, which cancels any remaining tasks in the scope if one of the tasks has failed. A shutdown-on-success policy is also available: it cancels any remaining tasks in the scope if one of the tasks has succeeded. It can be used to avoid doing unnecessary work when a successful result has already been achieved.

We can use a shutdown-on-success policy by calling an overload of the `StructuredTaskScope.open()` method that takes a `Joiner` as its parameter. Let's see what that would look like:

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

We've seen examples of two shutdown policies so far, but four more are provided out-of-the-box through the static factory methods in the `StructuredTaskScope.Joiner` interface. For example, `Joiner.allSuccessfulOrThrow()` will keep the scope alive until all subtasks have completed successfully, and cancels it if any subtasks fails. It's also possible to create your own shutdown policies by implementing the `Joiner` interface. That will allow you to have full control over when the scope will be shut down and what results will be collected.

#### What's Different From Java 26?

A few minor changes were made to the API compared to Java 26:

* The `StructuredTaskScope` and `Joiner` interfaces now have a third type parameter, `R_X`, to enable the caller to influence the exception type that the `join()` method of `StructuredTaskScope` can throw.
* A new static `open` method in `StructuredTaskScope` implements the default join policy and uses a given `UnaryOperator` to produce the `StructuredTaskScope` configuration. TODO: usecase?
* The `Joiner` factory methods `allSuccessfulOrThrow()`, `anySuccessfulOrThrow()`, and `awaitAllSuccessfulOrThrow()` now create joiners that cause `join()` to throw an `ExecutionException` when the outcome is an exception. New overloads of the three methods allow a `Function` to be specified to produce a different exception. TODO: explain usecase?
* The `Joiner` factory method `awaitAll()` has been removed. TODO: but why?
* The `onTimeout()` method of the `Joiner` interface has been replaced by the `timeout()` method, which either produces the result or throws an exception when the scope is cancelled by a timeout. If the `timeout()` method throws an exception then the exception is thrown with a `CancelledByTimeoutException` as the cause.

#### Preview Warning

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

[JEP 533](https://openjdk.org/jeps/533) has more details on the current state of this feature, should you wish to learn more.

### JEP 537: Vector API (Twelfth Incubator)

The Vector API makes it possible to express vector computations that reliably compile at runtime to optimal vector instructions. 
This means that these computations will significantly outperform equivalent scalar computations on the supported CPU architectures (x64 and AArch64).

#### Vector Computations? Help Me Out Here!

A *vector computation* is a mathematical operation on one or more one-dimensional matrices of an arbitrary length. Think of a vector as an array with a dynamic length. Furthermore, the elements in the vector can be accessed in constant time via indices, just like with an array. 

In the past, Java programmers could only program such computations at the assembly-code level. But now that modern CPUs support advanced [SIMD](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data) features (Single Instruction, Multiple Data), it becomes more important to take advantage of the performance gains that SIMD instructions and multiple lanes operating in parallel can bring. The Vector API brings that possibility closer to the Java programmer.

#### Code Example

Here is a code example (taken from the JEP) that compares a simple scalar computation over elements of arrays with its equivalent using the Vector API:

```java
void scalarComputation(float[] a, float[] b, float[] c) {
   for (int i = 0; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
   }
}

static final VectorSpecies<Float> SPECIES = FloatVector.SPECIES_PREFERRED;

void vectorComputation(float[] a, float[] b, float[] c) {
    int i = 0;
    int upperBound = SPECIES.loopBound(a.length);
    for (; i < upperBound; i += SPECIES.length()) {
        // FloatVector va, vb, vc;
        var va = FloatVector.fromArray(SPECIES, a, i);
        var vb = FloatVector.fromArray(SPECIES, b, i);
        var vc = va.mul(va)
                   .add(vb.mul(vb))
                   .neg();
        vc.intoArray(c, i);
    }
    for (; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
    }
}
```

From the perspective of the Java developer, this is just another way of expressing scalar computations. It might come across as being more verbose, but on the other hand it can bring spectacular performance gains. 

#### Typical Use Cases

The Vector API provides a way to write complex vector algorithms in Java that perform extremely well, such as vectorized `hashCode` implementations or specialized array comparisons. Numerous domains can benefit from this, including machine learning, linear algebra, encryption, text processing, finance, and code within the JDK itself.

#### What's Different From Java 26?

Compared to the eleventh incubator version of this feature in Java 26, no API changes were made.

The Vector API will keep incubating until necessary features of Project Valhalla become available as preview features. When that happens, the Vector API will be adapted to use them, and it will be promoted from incubation to preview.

#### More Information

For more information on this feature, read [JEP 537](https://openjdk.org/jeps/537).

### JEP 538: PEM Encodings of Cryptographic Objects (Third Preview)

Within a Java context, cryptographic objects such as public keys, private keys and certificates can be easily created and distributed. But outside of the Java world, the de facto standard is the [Privacy-Enhanced Mail](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) (PEM) format. Let's see an example of a PEM-encoded cryptographic object:

```
-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEi/kRGOL7wCPTN4KJ2ppeSt5UYB6u
cPjjuKDtFTXbguOIFDdZ65O/8HTUqS/sVzRF+dg7H3/tkQ/36KdtuADbwQ==
-----END PUBLIC KEY-----
```

The Java Platform currently doesn't include an easy-to-use API for decoding and encoding text in the PEM format, which means that decoding a PEM-encoded key can be a tedious job that involves careful parsing of the source PEM text. To further illustrate this point, encrypting and decrypting a private key currently requires over a dozen lines of code.

To solve this problem, JEP 538 introduces an API that can encode objects to the PEM format. It effectively acts as a bridge between Base64 and cryptographic objects. It involves a new interface and three new classes, in the `java.security` package:

[`BinaryEncodable`](https://cr.openjdk.org/~ascarpino/pem27/api/java.base/java/security/BinaryEncodable.html)
: A sealed interface that groups together all cryptographic objects that support converting their instances to and from byte arrays in the [Distinguished Encoding Rules](https://en.wikipedia.org/wiki/X.690#DER_encoding) (DER) format.

[`PEMEncoder`](https://cr.openjdk.org/~ascarpino/pem27/api/java.base/java/security/PEMEncoder.html)
: A class that declares methods for encoding `BinaryEncodable` objects into PEM text.

[`PEMDecoder`](https://cr.openjdk.org/~ascarpino/pem27/api/java.base/java/security/PEMDecoder.html)
: A class that declares methods for decoding PEM text to `BinaryEncodable` objects.

[`PEM`](https://cr.openjdk.org/~ascarpino/pem27/api/java.base/java/security/PEM.html)
: A class that implements `BinaryEncodable`, which can hold any type of PEM data. It allows you to encode and decode PEM tests yielding cryptographic objects for which no Java representation currently exists. 

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
String keyPairPem = pemEncoder.encodeToString(new KeyPair(publicKey, privateKey)); 

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

#### What's Different From Java 26?

A few minor changes were made to the API compared to Java 26:

* The `PEM` class is now an ordinary class rather than a record. It now includes constructors that accept Base64-encoded content in byte arrays, which is more convenient for some use cases.
* The `DEREncodable` interface is now named `BinaryEncodable`, to more accurately describe the binary data stored in PEM text.
* The `EncryptedPrivateKeyInfo` class now includes `getKeyPair` methods that decrypt PKCS#8-encoded text containing a `PublicKey`.
* The `getKey` and `getKeyPair` methods of `EncryptedPrivateKeyInfo` that took a password and `Provider` now take only a `Key`.
* The `withFactory` method of `PEMDecoder` is now named `withFactoriesOf` to better describe that key and certificate factories are obtained from the given `Provider`.
* A new `CryptoException` class indicates failures in cryptographic processing at runtime.

#### More Information

For more information on this feature, see [JEP 538](https://openjdk.org/jeps/538).

## New Defaults

Two JEPs in Java 27 make features that were introduced earlier the default.

### JEP 523: Make G1 the Default Garbage Collector in All Environments

TODO

#### What's Different From Java 26?

TODO

#### More Information

For more information on this feature, read [JEP TODO](https://openjdk.org/jeps/todo) or the [full feature description](https://hanno.codes/todo) from a previous article.

### JEP 536: JFR In-Process Data Redaction

TODO

#### What's Different From Java 26?

TODO

#### More Information

For more information on this feature, read [JEP TODO](https://openjdk.org/jeps/todo) or the [full feature description](https://hanno.codes/todo) from a previous article.

## Final thoughts

TODO
