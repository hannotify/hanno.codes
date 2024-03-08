---
layout: post
title: Java 22 TODO
date: 19-03-2024 04:30:00 +0200
header:
  teaser: /assets/images/blog/TODO.jpg
excerpt: TODO
tags: 
- java
---

TODO: intro

![TODO](/assets/images/blog/TODO.jpg)
> Image from <a href="https://TODO">TODO</a>

## From Project Amber

Java 22 contains four features that originated from [Project Amber](https://openjdk.org/projects/amber/):

* Statements before super(...)
* Unnamed Variables & Patterns
* String Templates
* Implicitly Declared Classes and Instance Main Methods

> The goal of Project Amber is to explore and incubate smaller, productivity-oriented Java language features.

### JEP 447: Statements before super(...) (Preview)

In Java, constructors run from top to bottom. On top of that, a superclass constructor must finish initializing its fields before a subclass constructor starts. This ensures proper object state initialization and prevents access to uninitialized fields. Java enforces this by requiring explicit constructor invocations to be the first statement in a constructor body, and constructor arguments cannot access the current object. While these rules ensure that constructors behave predictably, they may restrict the use of common programming patterns in constructor methods. The following code example illustrates this point:

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

This approach will be possible in Java 22, due to the introduction of _pre-construction contexts_.
Java used to treat the arguments to an explicit constructor invocation to be in a _static context_, as if they were in a static method. But this restriction is a bit stronger than necessary, which is why Java 22 introduces the aforementioned _pre-construction contexts_; a strictly weaker concept. It covers both the arguments to an explicit constructor invocation and any statements that occur before it. Within a pre-construction context, the rules are similar to normal instance methods, except that the code may not access the instance under construction.

#### What's Different From Java 21?

In Java 21 and earlier, statements were not allowed to appear before an explicit constructor invocation. In Java 22 such constructions are now valid, but only if these statements don't reference the instance being created.

Note that the JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to be able to take the feature for a spin.

#### More Information

The JEP contains a few more details on the specific rules that make up the restrictions of not being able to access the instance in a pre-construction context.
If you're interested in these details, please refer to [JEP 447](https://openjdk.org/jeps/447). 

Or if you want to try out 'statements before super(...)' for yourself, then here's a [GitHub repository](https://github.com/hannotify/java-22-release-day) to get you started.

### JEP 456: Unnamed Variables & Patterns

Data processing in Java has become increasingly streamlined since the introduction of [records](https://openjdk.org/jeps/395) and [record patterns](https://openjdk.org/jeps/440). But in some cases writing out an entire record pattern when some record components aren't even used in the logic that follows can be both cumbersome and confusing. Consider the following code example: 

```java
static boolean isDelayTimeEqualToReverbRoomSize(EffectLoop effectLoop) {
    if (effectLoop instanceof EffectLoop(Delay(int timeInMs), Reverb(String name, int roomSize))) {
        return timeInMs == roomSize;
    }
    return false;
}
```

This piece of code, which originates from a [music store example repository](https://github.com/hannotify/pattern-matching-music-store), deals with comparing two guitar effects that are stored in the `EffectLoop` that the fictional guitar player currently uses.  
Here, the logic in the `if` body doesn't reference the reverb name whatsoever, but for a long time Java didn't have a way to indicate this as an intentional omission. 
And so we've had no choice but to write the entire record pattern, leading future readers of this code to doubt the correctness of the implementation.

#### Unnamed Patterns

This situation was changed in Java 21, when _unnamed patterns_ became available through [JEP 443](https://openjdk.org/jeps/443), a feature that will be finalized in Java 22. Let's see how an unnamed pattern would change the example code we shared earlier:

```java
static boolean isDelayTimeEqualToReverbRoomSize(EffectLoop effectLoop) {
    if (effectLoop instanceof EffectLoop(Delay(int timeInMs), Reverb(_, int roomSize))) {
        return timeInMs == roomSize;
    }
    return false;
}
```

The underscore denotes the unnamed pattern here: it is an unconditional pattern which binds nothing. You can use it to indicate that it doesn't matter to what first value the pattern matches the `Reverb`, as long as the second parameter can be matched to an `int`.

#### Unnamed Pattern Variables

Java 21 also introduced _unnamed pattern variables_, a feature that'll also be finalized in Java 22. You can use an unnamed pattern variable whenever you care about the type your record pattern will match, but when you don't need any value bound to the pattern variable. Imagine our fictional guitar player want to use their tuner effect pedal to also support tuning notes played on a piano. In that case we could use unnamed pattern variables like this:

```java
static void apply(Effect effect, Piano piano) {
    System.out.println(switch(effect) {
        case Tuner(FlatNote _), Tuner(SharpNote _) -> "Tuning one of the black keys...";
        case Tuner(RegularNote _) -> "Tuning one of the white keys...";
        default -> "An unknown effect is active...";
    });
}
```

Here, we execute specific logic when we encounter a tuner that tunes a flat (♭) or sharp (♯) note. An unnamed pattern variable is the appropriate choice here, because the logic acts on the matched type only - meaning its value can be safely ignored.

#### Unnamed Variables

_Unnamed variables_ can be useful in situations where variables are unused and their names are irrelevant, for example when keeping a counter variable within the body of a for-each loop:

```java
int guitarCount = 0;
for (Guitar guitar : guitars) {
    if (guitarCount < LIMIT) { 
        guitarCount++;
    }
}
```

The `guitar` variable is declared and populated here, but it is never used. Unfortunately, its intentional non-use doesn't come across as such to the reader. Moreover, static code analysis tools like Sonar will probably complain about the unused variable, raising suspicions even more. Introducing an unnamed variable can better convey the intent of the code:

```java
int guitarCount = 0;
for (Guitar _ : guitars) {
    if (guitarCount < LIMIT) { 
        guitarCount++;
    }
}
```

Another good example could be handling exceptions in a generic way:

```java
var lesPaul = new Guitar("Les Paul");
try { 
    cart.add(stock.get(lesPaul, guitarCount));
} catch (OutOfStockException _) { 
    System.out.println("Sorry, out of stock!");
}
```

Keep in mind that unnamed variables only make sense when they're not visible outside a method, so they currently only work with local variables, exception parameters and lambda parameters. The theoretical concept of _unnamed method parameters_ is briefly touched upon in the JEP, but supporting that comes with enough challenges to at least warrant postponing it to a future JEP.

#### What's Different From Java 21?

Compared to the preview version of this feature in Java 21, nothing was changed or added. JEP 456 simply exists to finalize the feature.

#### More Information

For more information on this feature, see [JEP 456](https://openjdk.org/jeps/456).

### JEP 459: String Templates (Second Preview)

Several options in Java currently exist to compose a string from literal text and expressions:

* String concatenation with the `+` operator;
* `StringBuilder`;
* `String::format` or `String::formatted`;
* `java.text.MessageFormat`

However, these mechanisms come with drawbacks. They involve hard-to-read code (`+` operator) or verbose code (`StringBuilder`), they separate the input string from the parameters (`String::format`) or they require a lot of ceremony (`MessageFormat`).

_String interpolation_ is a mechanism that many programming languages offer as a solution to these drawbacks. But string interpolation comes with a drawback of its own: interpolated strings need to be manually validated by the developer to avoid dangerous risks like [SQL injection](https://en.wikipedia.org/wiki/SQL_injection).

This JEP re-previews the 'String Templates' feature: a template-based mechanism for composing strings that offers the benefits of interpolation, but would be less prone to introducing security vulnerabilities. A _template expression_ is a new kind of expression in Java, that can perform string interpolation but is also programmable in a way that helps developers compose strings safely and efficiently.

```java
String guitarType = "Les Paul";
System.out.println(STR."I bought a \{guitarType} yesterday.");
// outputs "I bought a Les Paul yesterday."
```

The template expression `STR."I bought a \{guitarType} yesterday."` consists of:

* A template processor (`STR`);
* A dot character, as seen in other kinds of expressions; and
* A template (`"I bought a \{guitarType} yesterday."`) which contains an embedded expression (`\{guitarType}`).

When a template expression is evaluated at run time, its template processor combines the literal text in the template with the values of the embedded expressions in order to produce a result. The embedded expressions can perform arithmetic, invoke methods and access fields: 

```java
int price = 12;
System.out.println(STR."A set of strings costs \{price} dollars; so each string costs \{price / 6} dollars.");
// outputs "A set of strings costs 12 dollars; so each string costs 2 dollars."
```

```java
record Guitar(String name, boolean inTune) {}
class GuitarTuner {
    public static void main(String... args) {
        var guitar = new Guitar("Gibson Les Paul Standard '50s Heritage Cherry Sunburst", false);
        System.out.println(STR."This guitar is \{guitar.inTune() ? "" : "not"} in tune.");
        // outputs "This guitar is not in tune.
    }
}
```

As you can see, double-quote characters can be used inside embedded expressions without escaping them as `\"`, making the switch from concatenation (using `+`) to template expressions easier. Multi-line template expressions are also possible; they use a syntax similar to that of [text blocks](https://docs.oracle.com/javase/specs/jls/se20/html/jls-3.html#jls-3.10.6):

```java
String title = "My Online Guitar Store";
String text = "Buy your next Les Paul here!";
String html = STR."""
        <html>
          <head>
            <title>\{title}</title>
          </head>
          <body>
            <p>\{text}</p>
          </body>
        </html>
        """;
```

#### Template Processors

`STR` is a template processor defined in the Java Platform. It performs string interpolation by replacing each embedded expression in the template with the (stringified) value of that expression. It is a `public static final` field that is automatically imported into every Java source file.

More template processors exist:

* `FMT` - besides performing interpolation, it also interprets format specifiers which appear to the left of embedded expressions. The format specifiers are the same as those defined in `java.util.Formatter`.
* `RAW` - a standard template processor that produces an unprocessed `StringTemplate` object.

#### Ensuring Safety

The construct `STR."..."` we've used so far is actually a short way to define a template and call its `process` method. That means that our first code example:

```java
String guitarType = "Les Paul";
System.out.println(STR."I bought a \{guitarType} yesterday.");
```

is equivalent to:

```java
String guitarType = "Les Paul";
StringTemplate template = RAW."I bought a \{guitarType} yesterday.");
System.out.println(STR.process(template));
```

Template expressions are designed to prevent the direct conversion of strings with embedded expressions to interpolated strings. This makes it impossible for potentially incorrect strings to spread. A template processor securely handles this interpolation, and if you forget to use one, the compiler will report an error.

```java
String guitarType = "Les Paul";
System.out.println("I bought a \{guitarType} yesterday."); // doesn't compile!
// outputs: "error: processor missing from template expression"
```

#### Custom Template Processors

Each template processor is an object that implements the functional interface `StringTemplate.Processor`, which means developers can easily create custom template processors. Custom template processors can make use of the methods `StringTemplate::fragments` and `StringTemplate::values` in order to use static fragments and dynamic values of the string template, respectively. 

Custom template processors can be useful for various use cases. Let's illustrate two of them with a few code examples:

```java
var JSON = StringTemplate.Processor.of(
    (StringTemplate st) -> new JSONObject(st.interpolate())
);

String name = "Gibson Les Paul Standard '50s Heritage Cherry Sunburst";
String type = "Les Paul";
JSONObject doc = JSON."""
    {
        "name": "\{name}",
        "type": "\{type}"
    };
    """;
```

So the `JSON` template processor returns instances of `JSONObject` instead of `String`.
If we wanted, we could simply add more validation logic to the implementation of `JSON` to make the template processor handles its parameters a bit more safely.

```java
record QueryBuilder(Connection conn) implements StringTemplate.Processor<PreparedStatement, SQLException> {
    public PreparedStatement process(StringTemplate st) throws SQLException {
        // 1. Replace StringTemplate placeholders with PreparedStatement placeholders
        String query = String.join("?", st.fragments());

        // 2. Create the PreparedStatement on the connection
        PreparedStatement ps = conn.prepareStatement(query);

        // 3. Set parameters of the PreparedStatement
        int index = 1;
        for (Object value : st.values()) {
            switch (value) {
                case Integer i -> ps.setInt(index++, i);
                case Float f   -> ps.setFloat(index++, f);
                case Double d  -> ps.setDouble(index++, d);
                case Boolean b -> ps.setBoolean(index++, b);
                default        -> ps.setString(index++, String.valueOf(value));
            }
        }

        return ps;
    }
}
```

```java
var DB = new QueryBuilder(conn);
String type = "Les Paul"; 
PreparedStatement ps = DB."SELECT * FROM Guitar g WHERE g.guitar_type = \{type}";
ResultSet rs = ps.executeQuery();
```

The `DB` custom template processor is capable of constructing `PreparedStatements` that have their parameters injected in a safe way.

#### What's Different From Java 21?

Except for a technical change in template expression types, there are no changes relative to the first preview.

TODO: this text was added to the JEP:
```
In a TemplateExpression, the type of TemplateProcessor must be a subtype of StringTemplate.Processor. The type of the template expression is the return type of the process(StringTemplate) method in the type of TemplateProcessor. If that method throws checked exceptions then the template expression must be wrapped in a try-catch block or the enclosing method must declare that it throws those exceptions; for more details, see below.
```
TODO: the "Alternatives" section in the JEP was changed. See in what way that should influence the section above.

Note that the JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to be able to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 459](https://openjdk.org/jeps/459).

### JEP 463: Implicitly Declared Classes and Instance Main Methods (Second Preview)

Java's take on the classic [Hello, World!](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program) program is notoriously verbose:

```java
public class HelloWorld { 
    public static void main(String[] args) { 
        System.out.println("Hello, World!");
    }
}
```

On top of that, it forces newcomers to Java to grasp a few concepts that they certainly don't need on their first day of Java programming:

* The `public` access modifier and the role it plays in encapsulating units of code, together with its counterparts `private`, `protected` and default;
* The `String[] args` parameter, that allows the operating system's shell to pass arguments to the program;
* The `static` modifier and how it's part of Java's class-and-object model.

The motivation for this JEP is to help programmers that are new to Java by introducing concepts in the right order, starting with the more fundamental ones. This is done by hiding the unnecessary details until they are useful in larger programs.

#### Changing the Launch Protocol

To achieve this, the JEP proposes the following changes to the launch protocol:

* allow _instance main methods_, which are not `static` and don't need a `public` modifier, nor a `String[]` parameter;

```java
class HelloWorld { 
    void main() { // this is an instance main method
        System.out.println("Hello, World!");
    }
}
```

* allow a compilation unit to implicitly declare a class:

```java
void main() { // this is an instance main method inside of an implicitly declared class
    System.out.println("Hello, World!");
}
```

#### A flexible launch protocol

Java 22 enhances the launch protocol to offer more flexibility in declaring a program's entry point. The `main` method of a launched class can now have `public`, `protected` or default access, for example. Other enhancements of the launch protocol include:

* If the launched class contains a main method with a String[] parameter then choose that method.
* Otherwise, if the class contains a main method with no parameters then choose that method.
* In either case, if the chosen method is static then simply invoke it.
* Otherwise, the chosen method is an instance method and the launched class must have a zero-parameter, non-private constructor. Invoke that constructor and then invoke the main method of the resulting object. If there is no such constructor then report an error and terminate.
* If there is no suitable main method then report an error and terminate.

#### Implicitly Declared Classes

With the introduction of implicitly declared classes, the Java compiler willconsider a method that is not enclosed in a class declaration, as well as any unenclosed fields and any classes declared in the file, to be members of an implicitly declared top-level class. 
Such a class always belongs to the unnamed package, is `final`, and can't implement interfaces or extend classes except `Object`. You can't reference it by name or use method references for its static methods, but you can use `this` and make method references to its instance methods. Implicitly declared classes can't be instantiated or referenced by name in code. They're mainly used as program entry points and must have a `main` method, enforced by the Java compiler.

#### What's Different From Java 21?

Java 22 contains the following changes compared to Java 21:

* The features that is now called _implicitly declared classes_ used to be called _unnamed classes_ in Java 21. The unnamed classes approach came with a mechanism to prevent an unnamed class being used by other classes. In contrast, implicitly declared classes use a simpler approach: a source file without an enclosing class declaration is said to implicitly declare a class with a name chosen by the host system. These classes behave like normal top-level classes and require no additional tooling, library, or runtime support.
* The procedure for selecting a main method to invoke was too complicated, taking into account both whether the method had a parameter and whether it was a static method or an instance method. Java 22 simplifies the selection process to two steps: If there is a candidate `main` method with a `String[]` parameter, then we invoke that method; otherwise we invoke a candidate `main` method with no parameters. There is no ambiguity here because a class cannot declare a static method and an instance method of the same name and signature.

Note that the JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to be able to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 463](https://openjdk.org/jeps/463).

## From Project Loom

Java 22 contains two features that originated from [Project Loom](http://openjdk.java.net/projects/loom/):

* Structured Concurrency
* Scoped Values

> Project Loom strives to simplify maintaining concurrent applications in Java by introducing *virtual threads* and an API for *structured concurrency*, among other things.

### JEP 462: Structured Concurrency (Second Preview)

Historically, Java's take on concurrency has been _unstructured_, meaning that tasks run independently of each other. They don't come with any hierarchy, scope, or other structure, which means they cannot easily pass errors or cancellation intent to each other.
To illustrate this, let's look at a code example that takes place in a restaurant:

> All code examples that illustrate Structured Concurrency were taken from my conference talk ["Java's Concurrency Journey Continues! Exploring Structured Concurrency and Scoped Values"](https://hanno.codes/talks/#javas-concurrency-journey-continues).

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

Now, consider the fact that the `announceCourse(..)` method in the `Waiter` class sometimes fails with an `OutOfStockException`, because one of the ingredients for the course might not be in stock. This can lead to some problems:

* If `zoe.announceCourse(CourseType.MAIN)` takes a long time to execute but `grover.announceCourse(CourseType.STARTER)` fails in the meantime, the `announceMenu(..)` method will unnecessarily wait for the main course announcement by blocking on `main.get()`, instead of cancelling it (which would be the sensible thing to do).
- If an exception happens in `zoe.announceCourse(CourseType.MAIN)`, `main.get()` will throw it, but `grover.announceCourse(CourseType.STARTER)` will continue to run in its own thread, resulting in thread leakage.
- If the thread executing `announceMenu(..)` is interrupted, the interruption will not propagate to the subtasks: all threads that run an `announceCourse(..)` invocation will leak, continuing to run even after `announceMenu()` has failed.

Ultimately the problem here is that our program is logically structured with task-subtask relationships, but these relationships exist only in the mind of the developer. We might all prefer structured code that reads like a sequential story, but this example simply doesn't meet that criterion.

In contrast, the execution of single-threaded code _always_ enforces a hierarchy of tasks and subtasks. Consider the following single-threaded version of our restaurant example:

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

So from these two examples it is evident that concurrent programming would be a lot easier and more intuitive if it would be able to enforce the hierarchy of tasks and subtasks, just like single-threaded code can.

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
At `2`, an `ExecutionException` can be thrown if an exception occurs in one of the threads. 
Once we reach `3`, we can be sure everything has gone well, and we can retrieve and process the results.

Actually, the main difference with the code we had before is the fact that we create threads (`fork`) within a new `scope`. 
Now we can be certain that the lifetimes of the three threads are confined to this scope, which coincides with the body of the try-with-resources statement.

Furthermore, we've gained _short-circuiting behaviour_. 
When one of the `announceCourse(..)` subtasks fails, the others are cancelled if they have not completed yet.
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

In this example the waiter is responsible for getting a valid `DrinkOrder` object based on the preferences of the guest and the current supply of drinks at the bar.
After the method `Waiter.getDrinkOrder(Guest guest, DrinkCategory... categories)` has been called, the waiter starts to list all available drinks in the drink categories that were passed to the method.
Once a guest hears something they like, they respond and the waiter creates a drink order. When this happens, the `getDrinkOrder(..)` method returns a `DrinkOrder` object and the scope will shut down. 
This means that any unfinished subtasks (such as the one in which Elmo is still listing different kinds of tea) will be cancelled.
The `result()` method at `1` will either return a valid `DrinkOrder` object, or throw an `ExecutionException` if one of the subtasks has failed.

#### Custom Shutdown Policies

Two shutdown policies are provided out-of-the-box, but it's also possible to create your own by extending the class `StructuredTaskScope` and its protected `handleComplete(..)` method.
That will allow you to have full control over when the scope will shut down and what results will be collected.

#### What's Different From Java 21?

Compared to the preview version of this feature in Java 21, nothing was changed or added. JEP 462 simply exists to gather more feedback from users of this feature.

Note that the JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to be able to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 462](https://openjdk.org/jeps/462).

### JEP 464: Scoped Values (Second Preview)


*Scoped values* enable the sharing of immutable data within and across threads.
They are preferred to thread-local variables, especially when using large numbers of virtual threads.

#### ThreadLocal

Since Java 1.2 we can make use of `ThreadLocal` variables, which confine a certain value to the thread that created it. Back then it could be a simple way to achieve thread-safety, [in some cases](https://stackoverflow.com/a/817926).

But thread-local variables also come with a few caveats. Every thread-local variable is mutable, which makes it hard to discern which component updates shared state and in what order. There's also the risk of memory leaks, because unless you call `remove()` on the `ThreadLocal` the data is retained until it is garbage collected (which is only after the thread terminates). And finally, thread-local variables of a parent thread can be inherited by child threads, which results in the child thread having to allocate storage for every thread-local variable previously written in the parent thread.

These drawbacks become more apparent now that virtual threads have been introduced, because millions of them could be active at the same time - each with their own thread-local variables - which would result in a significant memory footprint.

#### Scoped Values

Like a thread-local variable, a scoped value has multiple incarnations, one per thread. Unlike a thread-local variable, a scoped value is written once and is then immutable, and is available only for a bounded period during execution of the thread.

TODO: replace the JEP example with a Restaurant one.

The JEP illustrates the use of scoped values with the pseudo code example below:

```java
final static ScopedValue<...> V = ScopedValue.newInstance();

// In some method
ScopedValue.where(V, <value>)
           .run(() -> { ... V.get() ... call methods ... });

// In a method called directly or indirectly from the lambda expression
... V.get() ...
```

We see that `ScopedValue.where(...)` is called, presenting a scoped value and the object to which it is to be bound. The call to `run(...)` binds the scoped value, providing an incarnation that is specific to the current thread, and then executes the lambda expression passed as argument. During the lifetime of the `run(...)` call, the lambda expression, or any method called directly or indirectly from that expression, can read the scoped value via the value’s `get()` method. After the `run(...)` method finishes, the binding is destroyed.

#### Typical Use Cases

Scoped values will be useful in all places where currently thread-local variables are used for the purpose of one-way transmission of unchanging data. 

#### What's Different From Java 21?

Compared to the preview version of this feature in Java 21, nothing was changed or added. JEP 464 simply exists to gather more feedback from users of this feature.

Note that the JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to be able to take the feature for a spin.


#### More Information

For more information on this feature, see [JEP 464](https://openjdk.org/jeps/464).

## From Project Panama

Java 22 contains two features that originated from [Project Panama](http://openjdk.java.net/projects/panama/):

* Foreign Function & Memory API
* Vector API

> Project Panama aims to improve the connection between the JVM and foreign (non-Java) libraries.

### JEP 454: Foreign Function & Memory API


Java programs have always had the option of interacting with code and data outside of the Java runtime, through the [Java Native Interface](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/) (JNI).
And accessing foreign memory (outside of the JVM, so off-heap) was possible using either the [ByteBuffer API](https://docs.oracle.com/en/java/javase/19/docs/api/java.base/java/nio/ByteBuffer.html) or the [sun.misc.Unsafe API](https://github.com/openjdk/jdk/blob/master/src/jdk.unsupported/share/classes/sun/misc/Unsafe.java).

However, all these mechanisms have downsides, which is why a more modern API is now proposed to support foreign functions and foreign memory in a better way.

> Performance-critical libraries like [Tensorflow](https://github.com/tensorflow/tensorflow), [Lucene](https://lucene.apache.org/) or [Netty](https://netty.io/) typically rely on using foreign memory, because they need more control over the memory they use to prevent the cost and unpredictability that comes with garbage collection.

#### Code Example

In order to demonstrate the new API, [JEP 442](https://openjdk.org/jeps/442) lists a code example that obtains a method handle for a C library function `radixsort` and then uses it to sort four strings that start out as Java array elements:

```java
// 1. Find foreign function on the C library path
Linker linker          = Linker.nativeLinker();
SymbolLookup stdlib    = linker.defaultLookup();
MethodHandle radixsort = linker.downcallHandle(stdlib.find("radixsort"), ...);
// 2. Allocate on-heap memory to store four strings
String[] javaStrings = { "mouse", "cat", "dog", "car" };
// 3. Use try-with-resources to manage the lifetime of off-heap memory
try (Arena offHeap = Arena.ofConfined()) {
    // 4. Allocate a region of off-heap memory to store four pointers
    MemorySegment pointers
        = offHeap.allocate(ValueLayout.ADDRESS, javaStrings.length);
    // 5. Copy the strings from on-heap to off-heap
    for (int i = 0; i < javaStrings.length; i++) {
        MemorySegment cString = offHeap.allocateFrom(javaStrings[i]);
        pointers.setAtIndex(ValueLayout.ADDRESS, i, cString);
    }
    // 6. Sort the off-heap data by calling the foreign function
    radixsort.invoke(pointers, javaStrings.length, MemorySegment.NULL, '\0');
    // 7. Copy the (reordered) strings from off-heap to on-heap
    for (int i = 0; i < javaStrings.length; i++) {
        MemorySegment cString = pointers.getAtIndex(ValueLayout.ADDRESS, i);
        javaStrings[i] = cString.reinterpret(...).getString(0);
    }
} // 8. All off-heap memory is deallocated here
assert Arrays.equals(javaStrings, new String[] {"car", "cat", "dog", "mouse"});  // true
```

Let's look at some of the types this code uses in more detail to get a rough idea of their function and purpose within the Foreign Function & Memory API:

`Linker`
: Provides access to foreign functions from Java code, and access to Java code from foreign functions. It allows Java code to link against foreign functions, via *downcall method handles*. It also allows foreign functions to call Java method handles, via the generation of *upcall stubs*. See the [JavaDoc](https://download.java.net/java/early_access/jdk20/docs/api/java.base/java/lang/foreign/Linker.html) of this type for more information.

`SymbolLookup`
: Retrieves the address of a symbol in one or more libraries. See the [JavaDoc](https://download.java.net/java/early_access/jdk20/docs/api/java.base/java/lang/foreign/SymbolLookup.html) of this type for more information.

`Arena`
: Controls the lifecycle of memory segments. An arena has a scope, called the arena scope. When the arena is closed, the arena scope is no longer alive. As a result, all the segments associated with the arena scope are invalidated, their backing memory regions are deallocated (where applicable) and can no longer be accessed after the arena is closed. See the [JavaDoc](https://download.java.net/java/early_access/jdk20/docs/api/java.base/java/lang/foreign/Arena.html) of this type for more information.

`MemorySegment`
: Provides access to a contiguous region of memory. There are two kinds of memory segments: *heap segments* (inside the Java memory heap) and *native segments* (outside of the Java memory heap). See the [JavaDoc](https://download.java.net/java/early_access/jdk20/docs/api/java.base/java/lang/foreign/MemorySegment.html) of this type for more information. 

`ValueLayout`
: Models values of basic data types, such as *integral* values, *floating-point* values and *address* values. On top of that, it defines useful value layout constants for Java primitive types and addresses. See the [JavaDoc](https://download.java.net/java/early_access/jdk20/docs/api/java.base/java/lang/foreign/ValueLayout.html) of this type for more information.


#### What's Different From Java 21?

In Java 21, this feature was in third preview (in the form of [JEP 442](https://openjdk.org/jeps/442)) to gather more developer feedback. Based on this feedback the following changes happened in Java 22:

* A new linker option allowing clients to pass heap segments to downcall method handles is now provided;
* The Enable-Native-Access JAR-file manifest attribute has been introduced, allowing code in executable JAR files to call restricted methods without having to use the `--enable-native-access` command-line option;
* Clients to build C-language function descriptors programmatically have been enabled, avoiding platform-specific constants;
* Support for variable-length arrays in native memory has been improved;
* Support for arbitrary charsets for native strings has been added.

On top of that, the feature has now been finalized!

#### More Information

For more information on this feature, see [JEP 454](https://openjdk.org/jeps/454).

### JEP 460: Vector API (Seventh Incubator)

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

#### What's Different From Java 21?

Aside from a minor set of bugfixes and (performance) enhancements in the API, the biggest difference with Java 21 is the support for vector access with heap `MemorySegment`s that are backed by an array of any primitive element type. Previously access was limited to heap MemorySegments backed by an array of byte.

#### More Information

For more information on this feature, see [JEP 460](https://openjdk.org/jeps/460).

## HotSpot

Java 22 introduces a single change to [HotSpot](https://openjdk.org/groups/hotspot/):

* Region Pinning for G1

> The HotSpot JVM is the runtime engine that is developed by Oracle. It translates Java bytecode into machine code for the host operating system's processor architecture.

### JEP 423: Region Pinning for G1

TODO

#### What's Different From Java 21?

TODO

#### More Information

For more information on this feature, see [JEP 423](https://openjdk.org/jeps/423).

## Compiler

Java 22 also brings us a single addition that's part of the compiler:

* Launch Multi-File Source-Code Programs 
  
### JEP 458: Launch Multi-File Source-Code Programs

TODO

#### What's Different From Java 21?

TODO

#### More Information

For more information on this feature, see [JEP 458](https://openjdk.org/jeps/458).

## Core Libraries

Java 22 also brings you two additions that are part of the core libraries:

* Class-File API
* Stream Gatherers

### JEP 457: Class-File API (Preview)

TODO

#### What's Different From Java 21?

TODO

Note that the JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to be able to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 457](https://openjdk.org/jeps/457).

### JEP 461: Stream Gatherers (Preview)

TODO

#### What's Different From Java 21?

TODO

Note that the JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to be able to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 461](https://openjdk.org/jeps/461).

## Final thoughts

TODO: outro

TODO: proofreed the entire thing
