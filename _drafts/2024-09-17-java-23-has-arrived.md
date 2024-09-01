---
layout: post
title: Java 23 Has Arrived, And It Brings a Truckload of Features
date: 19-09-2024 04:30:00 +0200
header:
  teaser: /assets/images/blog/road-train.jpg
excerpt: TODO
tags: 
- java
---

TODO. It's been six months since Java 22 was released, so it's time for another fresh set of Java features. This post takes you on a tour of the JEPs that are part of this release, giving you a brief introduction to each of them. Where applicable the differences with Java 22 are highlighted and a few typical use cases are provided, so that you'll be more than ready to use these features after reading this!

![Java 23 brings a truckload of features](/assets/images/blog/road-train.jpg)
> Image from <a href="https://pxhere.com/en/photo/772190">PxHere</a>

## From Project Amber

Java 23 contains 4 features that originated from [Project Amber](https://openjdk.org/projects/amber/):

* Primitive Types in Patterns, instanceof, and switch (Preview)
* Module Import Declarations (Preview)
* Implicitly Declared Classes and Instance Main Methods (Third Preview)
* Flexible Constructor Bodies (Second Preview)

> The goal of Project Amber is to explore and incubate smaller, productivity-oriented Java language features.

### JEP 455: Primitive Types in Patterns, instanceof, and switch (Preview)

Support for pattern matching in Java has been increasing since Java 16, but its support for primitives was always limited to nested record patterns. This JEP proposes to support primitive types in all pattern contexts, and to extend `instanceof` and `switch` to work with all primitive types.

#### Pattern Matching for Switch

[Pattern matching for switch](https://openjdk.org/jeps/441) currently doesn't support type patterns that specify a primitive type. This JEP proposes to add support for primitive type patterns in `switch`. This would allow the following code example...

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

TODO

#### Pattern Matching for instanceof

TODO

#### Primitive Types in instanceof and switch

TODO

#### What's Different From Java 22?

TODO

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 455](https://openjdk.org/jeps/455).

### JEP 476: Module Import Declarations (Preview)

Many essential classes in the Java Class Library require imports to be usable.
In the following code, for example...

```java
import java.util.Map;                   // or import java.util.*;
import java.util.function.Function;     // or import java.util.function.*;
import java.util.stream.Collectors;     // or import java.util.stream.*;
import java.util.stream.Stream;         // 

String[] instruments = new String[] { "violin", "piano", "marimba" };
Map<String, String> m =
    Stream.of(instruments)
          .collect(Collectors.toMap(s -> s.toUpperCase().substring(0, 1), Function.identity()));
```

...the imports take up just as much space as the code itself.

Whether that's a bad thing or not depends on your import type preference.
In large, mature projects [single-type imports](https://google.github.io/styleguide/javaguide.html#s3.3-import-statements) are preferred by many.
However, in early-stage situations (when trying out a new feature, or when you're in [JShell](https://openjdk.org/jeps/222)) a more convenient approach could come in handy.

#### Module Import Declarations

This JEP introduces _module import declarations_, and they have the form `import module M;`.
A module import declarations imports all of the public top-level classes and interfaces in the packages exported by the module `M` to the current module. Any indirect exports by the module `M` will also be made available by a module import declaration.

For example, if you write `import module java.sql`, it means your class will be able to read classes and interfaces from the packages `java.sql` and `javax.sql` (because they are exported by the module [java.sql](https://docs.oracle.com/en/java/javase/22/docs/api/java.sql/module-summary.html#packages-summary)). On top of that, because java.sql pulls in three other modules transitively, any exported packages from those modules will also be available in your class.

#### Ambiguous Imports

When you import a module, it brings in several packages, which means you might end up importing classes that have the same simple name from different packages. This creates ambiguity, leading to a compile-time error.

In this source file, the simple name `Date` is ambiguous:

```java
import module java.base;      // exports java.util, which has a public Date class
import module java.sql;       // exports java.sql, which has a public Date class

...
Date d = ...                  // Error - Ambiguous name!
...
```

Resolving ambiguities is straightforward: use a single-type-import declaration. For example, to resolve the ambiguous `Date` of the previous example:

```java
import module java.base;      // exports java.util, which has a public Date class
import module java.sql;       // exports java.sql, which has a public Date class

import java.sql.Date;         // resolve the ambiguity of the simple name Date!

...
Date d = ...                  // Ok!  Date is resolved to java.sql.Date
...
```

#### What's Different From Java 22?

Up until Java 22, to import all public top-level classes and interfaces from a module, a wildcard-type import was required for each package in the module. In Java 23 the same behavior can be achieved using a single module import declaration.

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 476](https://openjdk.org/jeps/476).

### JEP 477: Implicitly Declared Classes and Instance Main Methods (Third Preview)

Java's take on the classic [Hello, World!](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program) program is notoriously verbose:

```java
public class HelloWorld { 
    public static void main(String[] args) { 
        System.out.println("Hello, World!");
    }
}
```

On top of that, it forces newcomers to grasp a few concepts that they certainly don't need on their first day of Java programming:

* The `public` access modifier and its role in encapsulating units of code, together with its counterparts `private`, `protected` and default;
* The `String[] args` parameter, that allows the operating system's shell to pass arguments to the program;
* The `static` modifier and how it's part of Java's class-and-object model;
* How to import basic utility classes, to prevent the need for the mysterious `System.out.println` incantation.

The motivation for this JEP is to help programmers that are new to Java by introducing concepts in the right order, starting with the more fundamental ones. This is done by hiding any unnecessary details until they are useful in larger programs.

#### Proposed Changes

To achieve this, the JEP proposes the following changes:

* change the launch protocol to allow _instance main methods_, which are not `static` and don't need a `public` modifier, nor a `String[]` parameter;

```java
class HelloWorld { 
    void main() { // this is an instance main method
        System.out.println("Hello, World!");
    }
}
```

* allow a compilation unit to implicitly declare a class:

```java
void main() { // this is an instance main method in an implicitly declared class
    System.out.println("Hello, World!");
}
```

* in implicitly declared classes, automatically import useful methods for textual input and output, thereby avoiding the mysterious `System.out.println`:

```java
void main() {
    println("Hello, World!"); // this method is automatically imported
}
```

#### A Flexible Launch Protocol

Java 23 enhances the launch protocol to offer more flexibility in declaring a program's entry point. The `main` method of a launched class can now have `public`, `protected` or default access. Other enhancements of the launch protocol include:

* If the launched class contains a main method with a `String[]` parameter, then choose that method.
* Otherwise, if the class contains a main method with no parameters, then choose that method.
* In either case, if the chosen method is `static`, then simply invoke it.
* Otherwise, the chosen method is an instance method and the launched class must have a zero-parameter, non-private constructor. Invoke that constructor and then invoke the `main` method of the resulting object. If there is no such constructor, then report an error and terminate.
* If there is no suitable `main` method, then report an error and terminate.

#### Implicitly Declared Classes

With the introduction of implicitly declared classes, the Java compiler will consider a method that is not enclosed in a class declaration, as well as any unenclosed fields and any classes declared in the file, to be members of an implicitly declared top-level class. 
Such a class always belongs to the unnamed package, is `final`, and can't implement interfaces or extend classes except `Object`. You can't reference it by name or use method references for its static methods, but you can use `this` and make method references to its instance methods. Implicitly declared classes can't be instantiated or referenced by name in code. They're mainly used as program entry points and must have a `main` method, enforced by the Java compiler.

#### Interacting with the Console

Many beginner programs need to interact with the console, for example:

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

class Main {
    public static void main(String... args) {
        System.out.println("Please enter your name:");
        String name = "";
        try {
            BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
            name = reader.readLine();
        } catch (IOException ioe) {
            ioe.printStackTrace();
        }
        System.out.println("Hello, " + name);
    }
}
```

The code may seem familiar to experienced developers, but beginners often find it confusing due to new concepts like `import`, `try`, `catch`, `BufferedReader` vs. `InputStreamReader` and `IOException`. While there are other approaches, none are significantly better for those just starting out.

To make interacting with the console easier for beginners, implicit classes now automatically import all static methods from the new class [`java.io.IO`](https://cr.openjdk.org/~prappo/8305457/java.base/java/io/IO.html):

* `public static void println(Object obj);`
* `public static void print(Object obj);`
* `public static String readln(String prompt);`

Combined with the other features from this JEP, the code above can now be simplified to:

```java
void main() {
    String name = readln("Please enter your name:");
    print("Hello, ");
    println(name);
}
```

#### Automatic Import of the `java.base` Module

To make writing small programs easier, all public top-level classes and interfaces from the `java.base` module are now automatically available in every implicit class, as if they were imported. 
This means popular APIs from packages like `java.io`, `java.math`, and `java.util` can be used right away, eliminating the need for explicit import statements that might confuse beginners.

> This implicit behaviour can also be made explicit to make it work in any other class, by using a [module import declaration](#jep-476-module-import-declarations-preview) like `import module java.base`, which can be helpful when converting an implicit class to a top-level class.

#### What's Different From Java 22?

Java 23 contains the following changes compared to Java 22:

* Implicitly declared classes automatically import three static methods for simple textual I/O with the console. These methods are declared in the new top-level class [java.io.IO](https://cr.openjdk.org/~prappo/8305457/java.base/java/io/IO.html).
* Implicitly declared classes automatically import, on demand, all of the public top-level classes and interfaces of the packages exported by the `java.base` module.

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 477](https://openjdk.org/jeps/477).

### JEP 482: Flexible Constructor Bodies (Second Preview)

Suppose we have a superclass `Orchestra` and a subclass `StringQuartet`. The constructors of these classes must work together to ensure a valid instance. The `StringQuartet` constructor is responsible for any `StringQuartet`-specific fields, while the `Orchestra` constructor is responsible for fields declared in the `Orchestra` class. Since code in the `StringQuartet` constructor might refer to fields initialized by the `Orchestra` constructor, the latter must run first.

This example illustrates that constructors should run from top to bottom. This also means that a superclass constructor must finish initializing its fields before a subclass constructor starts. This ensures proper object state initialization and prevents access to uninitialized fields. Java enforces this by requiring explicit constructor calls (like `super(...)` or `this(...)`) to be the first statement in a constructor body, and constructor arguments cannot access the current object. While these rules ensure that constructors behave predictably, they may restrict the use of common programming patterns in constructor methods. The following code example illustrates this point:

```java
class StringQuartet extends Orchestra {
    public StringQuartet(List<Instrument> instruments) {
        super(instruments); // Potentially unnecessary work!

        if (instruments.size() != 4) {
            throw new IllegalArgumentException("Not a quartet!");
        }
    }
}
```

It would be better to let the constructor fail fast, by validating its arguments before the `super(...)` constructor is called.
Pre-Java 22, we could only achieve this by introducing a `static` method that acts upon the value passed to the super constructor.

```java
public class StringQuartet extends Orchestra {
    public StringQuartet(List<Instrument> instruments) {
        super(validate(instruments));
    }

    private static List<Instrument> validate(List<Instrument> instruments) {
        if (instruments.size() != 4) {
            throw new IllegalArgumentException("Not a quartet!");
        }
    }
}
```

But a far more readable way to write the same would be:

```java
public class StringQuartet extends Orchestra {
    public StringQuartet(List<Instrument> instruments) {
        if (instruments.size() != 4) {
            throw new IllegalArgumentException("Not a quartet!");
        }

        super(instruments);
    }
}
```

This approach will be possible in Java 23, due to the introduction of _early construction contexts_.
Java used to treat the arguments to an explicit constructor invocation to be in a _static context_, as if they were in a static method. But this restriction is a bit stronger than necessary, which is why Java 23 introduces the aforementioned _early construction contexts_; a strictly weaker concept. It covers both the arguments to an explicit constructor invocation and any statements that occur before it. Code in an early construction context must not use the instance under construction, except to initialize fields that do not have their own initializers.

#### What's Different From Java 22?

* The feature was renamed: in Java 22 it was known as "Statements before super(...)";
* A constructor body is now allowed to initialize fields in the same class before explicitly invoking a constructor. This possibility prevents any superclass constructor from seeing the default value of a subclass field (like `0`, `false` or `null`). 

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 482](https://openjdk.org/jeps/482).

### What Happened to String Templates?

Java 22 had a feature called [string templates](https://hanno.codes/2024/03/19/java-22-is-here/#jep-459-string-templates-second-preview), which was in second preview back then. 
However, the feature is _not_ included in Java 23, which might be surprising, but at the same time it's a valid possibility.
We have always known that [preview features are impermanent](https://openjdk.org/jeps/12), and might even be removed instead of promoted to 'final' status.

So what happened? Well, in short: the effort did not meet the needs of the developers.
Like any preview feature, feedback was received from the developer community, and quite a few issues were raised.
Some remarks were about syntax, and others were about nested templates.
But the most relevant feedback was about the co-dependence of _string templates_ and _template processors_.
To make the feature perform better than `String.format(...)`, it was crucial that the template processor would appear close to the string template it had to process, which is why the special syntax `PROCESSOR."template"` was introduced.
Based on the feedback, on the [Project Amber mailing list](https://mail.openjdk.org/pipermail/amber-spec-observers/2024-March/thread.html#4230)  a discovery was made: the template and the code processing it didn't _have_ to appear side by side to achieve a performance boost. This meant that the special syntax could also potentially be dropped.
This realization ultimately sent the feature back to the drawing board! 
While the new direction was taking shape, the Java 23 deadline was fast approaching and the choice was made to drop the feature entirely, instead of repreviewing it unchanged while everyone knew the design would change drastically in the future.

#### What Happens Next?

So, what happens next? Well, a new JEP will probably appear in the future, but it might take a while, as much of the initial design has to be reconsidered.
But based on the above template processors probably won't have a prominent place.
And string templates will probably become a new kind of literal (similar to String literals), because there's no need for them to be tied to their template processor any more.
There's no way for us to know exactly when a new proposal will be ready, so we'll just have to wait and see.
And in the meantime, if you're interested, [this excellent video](https://inside.java/2024/06/20/newscast-71) by Nicolai Parlog has some more interesting details to keep you occupied.

## From Project Loom

Java 23 contains 2 features that originated from [Project Loom](http://openjdk.java.net/projects/loom/):

* Structured Concurrency (Third Preview)
* Scoped Values (Third Preview)

> Project Loom strives to simplify maintaining concurrent applications in Java by introducing *virtual threads* and an API for *structured concurrency*, among other things.

### JEP 480: Structured Concurrency (Third Preview)

Java's take on concurrency has always been _unstructured_, meaning that tasks run independently of each other. There's no hierarchy, scope, or other structure involved, which means errors or cancellation intent is hard to communicate.
To illustrate this, let's look at a code example that takes place in a restaurant:

> The code examples that illustrate Structured Concurrency were taken from my conference talk ["Java's Concurrency Journey Continues! Exploring Structured Concurrency and Scoped Values"](https://hanno.codes/talks/#javas-concurrency-journey-continues).

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

Let's now take a look at a structured, concurrent version of our example:

```java
public class StructuredConcurrencyRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            Supplier<Course> starter = scope.fork(() -> grover.announceCourse(CourseType.STARTER));
            Supplier<Course> main = scope.fork(() -> zoe.announceCourse(CourseType.MAIN));
            Supplier<Course> dessert = scope.fork(() -> rosita.announceCourse(CourseType.DESSERT));

            scope.join(); // 1
            scope.throwIfFailed(); // 2

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get()); // 3
        }
    }
}
```
The scope's purpose is to keep the threads together.
At `1`, we wait (`join`) until _all_ threads are done with their work. 
If one of the threads is interrupted, an `InterruptedException` is thrown here. 
At `2`, an `ExecutionException` is thrown if an exception occurs in one of the threads. 
Once we reach `3`, we can be sure everything has gone well, and we can retrieve and process the results.

Actually, the main difference with the code we had before is the fact that we create threads (`fork`) within a new `scope`. 
Now we can be certain that the lifetimes of the three threads are confined to this scope, which coincides with the body of the try-with-resources statement.

Furthermore, we've gained _short-circuiting behaviour_. 
When one of the `announceCourse(..)` subtasks fails, the others are cancelled if they didn't complete yet.
This behaviour is managed by the `ShutdownOnFailure` policy.
We've also gained _cancellation propagation_.
When the thread that runs `announceMenu()` is interrupted before or during the call to `scope.join()`, all subtasks are cancelled automatically when the thread exits the scope.

#### Shutdown on Success

A shutdown-on-failure policy cancels tasks if one of them fails, while a _shutdown-on-success_ policy cancels tasks if one succeeds. The latter is useful to prevent unnecessary work once a successful result is obtained.

Let's see what a shutdown-on-success implementation would look like:

```java
public record DrinkOrder(Guest guest, Drink drink) {}

public class StructuredConcurrencyBar implements Bar {
    @Override
    public DrinkOrder determineDrinkOrder(Guest guest) throws InterruptedException, ExecutionException {
        Waiter zoe = new Waiter("Zoe");
        Waiter elmo = new Waiter("Elmo");

        try (var scope = new StructuredTaskScope.ShutdownOnSuccess<DrinkOrder>()) {
            scope.fork(() -> zoe.getDrinkOrder(guest, BEER, WINE, JUICE));
            scope.fork(() -> elmo.getDrinkOrder(guest, COFFEE, TEA, COCKTAIL, DISTILLED));

            return scope.join().result(); // 1
        }
    }
}
```

In this example the waiter is responsible for getting a valid `DrinkOrder` object based on guest preference and the drinks supply at the bar.
In the method `Waiter.getDrinkOrder(Guest guest, DrinkCategory... categories)`, the waiter starts to list all available drinks in the drink categories that were passed to the method.
Once a guest hears something they like, they respond and the waiter creates a drink order. When this happens, the `getDrinkOrder(..)` method returns a `DrinkOrder` object and the scope will shut down. 
This means that any unfinished subtasks (such as the one in which Elmo is still listing different kinds of tea) will be cancelled.
The `result()` method at `1` will either return a valid `DrinkOrder` object, or throw an `ExecutionException` if one of the subtasks has failed.

#### Custom Shutdown Policies

Two shutdown policies are provided out-of-the-box, but it's also possible to create your own by extending the class `StructuredTaskScope` and its protected `handleComplete(..)` method.
That will allow you to have full control over when the scope will shut down and what results will be collected.

#### What's Different From Java 22?

Compared to the preview version of this feature in Java 22, nothing was changed or added. JEP 480 simply exists to gather more feedback from users.

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 480](https://openjdk.org/jeps/480).

### JEP 481: Scoped Values (Third Preview)

*Scoped values* enable the sharing of immutable data within and across threads.
They are preferred to thread-local variables, especially when using large numbers of virtual threads.

> The code examples that illustrate Scoped Values were taken from my conference talk ["Java's Concurrency Journey Continues! Exploring Structured Concurrency and Scoped Values"](https://hanno.codes/talks/#javas-concurrency-journey-continues).

#### ThreadLocal

Since Java 1.2 we can make use of `ThreadLocal` variables, which confine a certain value to the thread that created it. Back then this was a simple way to achieve thread-safety, [in some cases](https://stackoverflow.com/a/817926).

But thread-local variables also come with a few caveats. Every thread-local variable is mutable, making it hard to discern where shared state is updated and in what order. There's also the risk of memory leaks, because unless you call `remove()` on the `ThreadLocal` the data is retained until it is garbage collected (which is only after the thread terminates). And finally, thread-local variables of a parent thread can be inherited by child threads, meaning that the child thread has to allocate storage for every thread-local variable previously written in the parent thread.

These drawbacks become more apparent now that virtual threads have been introduced, because millions of them could be active at the same time - each with their own thread-local variables - resulting in a significant memory footprint.

#### Scoped Values

Like a thread-local variable, a scoped value has multiple incarnations, one per thread. Unlike a thread-local variable, a scoped value is written once and is then immutable. It's available only for a bounded period during execution of the thread.

To demonstrate this feature, we've added a scoped value to the `StructuredConcurrencyBar` class that you're already familiar with:

```java
public class StructuredConcurrencyBar implements Bar {
    private static final ScopedValue<Integer> drinkOrderId = ScopedValue.newInstance();

    @Override
    public DrinkOrder determineDrinkOrder(Guest guest) throws Exception {
        Waiter zoe = new Waiter("Zoe");
        Waiter elmo = new Waiter("Elmo");

        return ScopedValue.where(drinkOrderId, 1)
                .call(() -> {
                    try (var scope = new StructuredTaskScope.ShutdownOnSuccess<DrinkOrder>()) {
                        scope.fork(() -> zoe.getDrinkOrder(guest, BEER, WINE, JUICE));
                        scope.fork(() -> elmo.getDrinkOrder(guest, COFFEE, TEA, COCKTAIL, DISTILLED));

                        return scope.join().result();
                    }
                });
    }
}
```

We see that `ScopedValue.where(...)` is called, presenting a scoped value and the object to which it is to be bound. The invocation of `call(...)` binds the scoped value, providing an incarnation that is specific to the current thread, and then executes the lambda expression passed as argument. During the lifetime of `call(...)`, the lambda expression - and any method called (in)directly from it - can read the scoped value via the value’s `get()` method. After the `call(...)` method finishes, the binding is destroyed.

#### Typical Use Cases

Scoped values will be useful in all places where currently thread-local variables are used for the purpose of one-way transmission of unchanging data. 

#### What's Different From Java 22?

The method `ScopedValue.callWhere(key, value, operation)` (which is a shorter way to write `ScopedValue.where(key, value).call(operation);`) can now throw a checked exception of a generic type. This means that you're now able to catch that exception like this:

```java
    try {
        var result = ScopedValue.callWhere(X, "hello", () -> bar());
    } catch (Exception e) {
        handleFailure(e);
    }
    // ...
```

Note that this JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 481](https://openjdk.org/jeps/481).

## HotSpot

Java 23 introduces a single change to [HotSpot](https://openjdk.org/groups/hotspot/):

* ZGC: Generational Mode by Default

> The HotSpot JVM is the runtime engine that is developed by Oracle. It translates Java bytecode into machine code for the host operating system's processor architecture.

### JEP 474: ZGC: Generational Mode by Default

The Z Garbage Collector (ZGC) is a scalable, low-latency garbage collector. It has been [available for production use since Java 15](https://openjdk.org/jeps/377) and has been designed to keep pause times consistent and short, even for very large heaps. It uses techniques like region-based memory management and compaction to achieve this.

Java 21 introduced [an extension to ZGC](https://openjdk.org/jeps/439) that maintains separate *generations* for young and old objects, allowing ZGC to collect young objects (which tend to die young) more frequently. This will result in a significant performance gain for applications running with generational ZGC, without sacrificing any of the valuable properties that the Z garbage collector is already known for.

In Java 21, a command-line option was required to enable generational mode. Java 23 makes generational mode the default mode for ZGC, whereas non-generational mode has been deprecated. It will probably be removed in a future Java version.

#### What's Different From Java 22?

To run your workload with Generational ZGC in Java 22, the following configuration was needed:

```bash
$ java -XX:+UseZGC -XX:+ZGenerational ...
```

In Java 23, this command-line option is no longer needed, it will use generational mode by default:

```bash
$ java -XX:+UseZGC ...
```

When you *do* still include it, a warning will be issued that the `ZGenerational` option is deprecated.

To revert back to non-generational ZGC, you would need to run:

```bash
$ java -XX:+UseZGC -XX:-ZGenerational ...
```

And you would get two warnings:
* The `ZGenerational` option is deprecated;
* Non-generational mode is deprecated.

#### More Information

For more information on this feature, see [JEP 474](https://openjdk.org/jeps/474).
  
## Core Libraries

Java 23 also brings you 4 additions that are part of the core libraries:

* Class-File API (Second Preview)
* Vector API (Eighth Incubator)
* Stream Gatherers (Second Preview)
* Deprecate the Memory-Access Methods in sun.misc.Unsafe for Removal

### JEP 466: Class-File API (Second Preview)

TODO

#### What's Different From Java 22?

TODO

#### More Information

TODO

### JEP 469: Vector API (Eighth Incubator)

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

#### What's Different From Java 22?

Compared to the seventh incubator version of this feature in Java 22, nothing was changed or added.
The Vector API will incubate until necessary features of Project Valhalla become available as preview features.
When that happens, the Vector API will be adapted to use them, and it will be promoted from incubation to preview.

#### More Information

For more information on this feature, see [JEP 469](https://openjdk.org/jeps/469).

### JEP 473: Stream Gatherers (Second Preview)

TODO

#### What's Different From Java 22?

TODO

#### More Information

TODO

### JEP 471: Deprecate the Memory-Access Methods in sun.misc.Unsafe for Removal

TODO

#### What's Different From Java 22?

TODO

#### More Information

TODO

## Tools

Java 23 contains a single feature that is part of the JavaDoc tool:

* Markdown Documentation Comments

### JEP 467: Markdown Documentation Comments

The JavaDoc documentation standard has been part of Java since its inception in 1995.
Back then, HTML was the obvious choice when JavaDoc needed a way to support basic markup.
Nowadays HTML is far less likely to be manually produced by humans, but rather generated from some other markup language that is more suitable for humans.
Coupled with the fact that inline JavaDoc tags (like `{@link}` and `{@code}`) are not that familiar to developers, it makes perfect sense that Java's documentation tool will start supporting Markdown!

Markdown documentation comments use the `///` prefix instead of the familiar `/**` one. This is due to two reasons:

1. Block comments that begin with `/*` cannot contain the `*/` character sequence. This makes it currently very difficult to include code examples that contain embedded `/* ... */` comments. In `//` comments there's no such restriction.
2. A traditional documentation comment (starting with `/**`) doesn't require the leading whitespace and the asterisk character on each line. In these cases, Markdown constructs that uses asterisks (like emphasis or list items) would clash with the traditional documentation comment syntax.

#### Example of the Differences

To illustrate how documentation comments can change now that Markdown is supported, here's an example screenshot from the JEP: 

![Differences between regular documentation comment and Markdown documentation comment](https://cr.openjdk.org/~jjg/Object-hashcode-diff-3.png)

#### Syntax

Markdown documentation comments are written in the [CommonMark](https://spec.commonmark.org/0.30) variant of Markdown. Simple tables are supported, using the syntax of [GitHub Flavored Markdown](https://github.github.com/gfm/). For example:

```
/// | Make     | Model      |
/// |----------|------------|
/// | Fender   | Telecaster |
/// | Gibson   | Les Paul   |
/// | Epiphone | Casino     |
```

You can create a link to an element declared elsewhere in your API, by simply enclosing a reference to the element in square brackets. 
For example, to link to `java.util.List`, you can write `[java.util.List]`, or just `[List]`.
The link is equivalent to using the standard JavaDoc `{@link ...}` tag.

You can link to any kind of program element:

```
/// - a module [com.hannotify.guitarstore/]
/// - a package [com.hannotify.guitarstore.products]
/// - a class [Guitar]
/// - a field [Guitar#name]
/// - a method [Guitar#isInTune()]
```

To create a link with alternative text, you can use the form [text][element]. For example:

```
/// Please feel free to use our [utility guitar tuning method](Guitar#isInTune())!
```

#### What's Different From Java 22?

In Java 22, it wasn't yet possible to include Markdown in documentation comments. This possibility has been added in Java 23.

#### More Information

For more information on this feature, see [JEP 467](https://openjdk.org/jeps/467).

## Final thoughts

It seems clear to me that Java 23 is TODO, with no less than 12 JEPs delivered! And that's not even all that's new: [many other updates](https://jdk.java.net/22/release-notes) were included in this release, including thousands of performance, stability and security updates. TODO