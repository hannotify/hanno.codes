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

### JEP 441: Pattern Matching for switch

The feature 'Pattern Matching for switch' that was first introduced in Java 17 has reached completion status, now that Java 21 has been released.

Since Java 16 we have been able to avoid casting after `instanceof` checks by using 'Pattern Matching for instanceof'. Let's see how that works in a code example.

> All code examples that illustrate Project Amber features were taken from my conference talk ["Pattern Matching: Small Enhancement or Major Feature?"](https://hanno.codes/talks/#pattern-matching-small-enhancement-or-major-feature).

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

For more information on this feature, see [JEP 441](https://openjdk.org/jeps/441). Or if you want to try out pattern matching for switch and you liked the code examples, then here's a [GitHub repository](https://github.com/hannotify/pattern-matching-music-store) to get you started.

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

Apart from some minor editorial changes, the main change since the second preview is the removal of support for record patterns appearing in the header of an enhanced for statement. Java 20 [introduced the feature](https://hanno.codes/2023/03/21/java-20-release-day-heres-whats-new/#enhanced-for-statements), but it was dropped in Java 21 because the feature "may need a significant redesign, to better align with other features under consideration" (see [this OpenJDK ticket](https://bugs.openjdk.org/browse/JDK-8304401) for more information). This may seem strange, but that is the way it can be with preview features: they are fully specified and implemented, and yet impermanent until the feature has been fully delivered. Not to worry though, JEP 440 also hints that support for record patterns in enhanced for statements may be re-proposed in a future JEP, presumably when able to be better aligned with other features under consideration.

#### More Information

For more information on this feature, see [JEP 440](https://openjdk.org/jeps/440). Or if you want to try out record patterns and you liked the code examples, then here's a [GitHub repository](https://github.com/hannotify/pattern-matching-music-store) to get you started.

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

### JEP 430: String Templates (Preview)

There are currently multiple ways in Java to compose a string from literal text and expressions:

* String concatenation with the `+` operator;
* `StringBuilder`;
* `String::format` or `String::formatted`;
* `java.text.MessageFormat`

However, these mechanisms come with a few drawbacks. They involve hard-to-read code (`+` operator) or verbose code (`StringBuilder`), they separate the input string from the parameters (`String::format`) or they require a lot of ceremony (`MessageFormat`).

_String interpolation_ is a mechanism that many programming languages offer as a solution to these drawbacks. But string interpolation comes with a drawback of its own: interpolated strings (typically holding HTML/XML documents, JSON snippets or SQL statements) need to be manually validated by the developer to avoid dangerous risks like [SQL injection](https://en.wikipedia.org/wiki/SQL_injection).

This JEP proposes the 'String Templates' feature: a first-class, template-based mechanism for composing strings that offers the benefits of interpolation, but would be less prone to introducing security vulnerabilities. A _template expression_ is a new kind of expression in Java, that can perform string interpolation but is also programmable in a way that helps developers compose strings safely and efficiently.

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

#### What's Different From Java 20?

Java 20 didn't contain anything related to string templates yet, so Java 21 is the first time we get to experiment with them. Note that the JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to be able to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 430](https://openjdk.org/jeps/430).

## From Project Loom

Java 21 contains three features that originated from [Project Loom](http://openjdk.java.net/projects/loom/):

* Virtual Threads;
* Structured Concurrency.
* Scoped Values;

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

### JEP 453: Structured Concurrency (Preview)

Java's current implementation of concurrency is _unstructured_, meaning that tasks run independently of each other. They don't come with any hierarchy, scope, or other structure, which means they cannot easily pass errors or cancellation intent to each other.
To illustrate this, let's look at a code example that takes place in a restaurant:

> All code examples that illustrate Structured Concurrency or Scoped Values were taken from my conference talk ["Java's Concurrency Journey Continues! Exploring Structured Concurrency and Scoped Values"](https://hanno.codes/talks/#javas-concurrency-journey-continues-exploring-structured-concurrency-and-scoped-values).

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

#### What's Different From Java 20?

The situation is roughly the same as how it was in Java 20 (see [JEP 437](https://openjdk.org/jeps/437)), except for two minor changes:

* Structured Concurrency is now a Preview API;
* The `StructuredTaskScope::fork` method now returns a `Subtask` instead of a `Future` ([why?](https://openjdk.org/jeps/453#Why-doesn't-fork----return-a-Future?)).

Note that the JEP is in the [preview](https://openjdk.org/jeps/12) stage, so you'll need to add the `--enable-preview` flag to the command-line to be able to take the feature for a spin.

#### More Information

For more information on this feature, see [JEP 453](https://openjdk.org/jeps/453). Or if you want to try out structured concurrency and you liked the code examples, then here's a [GitHub repository](https://github.com/hannotify/structured-concurrency-bar) to get you started.

### JEP 446: Scoped Values (Preview)

TODO

#### What's Different From Java 20?

TODO

#### More Information

For more information on this feature, see [JEP 446](https://openjdk.org/jeps/446).

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
This currently means Windows x86-32 users can still use JDK 21; however a future release will actually remove the support, and by that time the affected users are expected to have migrated to Windows x64 and a 64-bit JVM. 

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

One of the most well-known APIs in Java is the [collections framework](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/util/doc-files/coll-index.html), but did you know it doesn't even contain a collection type that represents an ordered sequence of elements? Sure, there are some subtypes that support encounter order (such as `List`, `SortedSet` and `Deque`), but their common supertype is `Collection`, which does not support it. So support for encounter order is dispersed across the type hierarchy, and that leads to more challenges. Consider obtaining the first or last element of a collection, for instance. While numerous implementations accommodate this functionality, each adopts a distinct approach, and a few are not as obvious (or don't support it at all). The process of sequentially traversing collection elements from the beginning to the end might appear simple and systematic, but iterating in reverse order is notably less straightforward.

It seems like the concept of a collection with defined encounter order exists in multiple places in the collections framework, but there is no single type that represents it. And this is the gap in the collections framework that will be filled by Sequenced Collections.

#### New Interfaces

The JEP will introduce new interfaces for *sequenced collections*, *sequenced sets* and *sequenced maps*. They are retrofitted into the existing collections type hierarchy (making ample use of [default methods](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)), as follows:

![A diagram of how the new interfaces related to sequence collections are retrofitted into the existing collections type hierarchy.](/assets/images/blog/java-21-release-day/sequenced-collections-diagram.png)
> Image from <a href="https://openjdk.org/jeps/431#Retrofitting">JEP 431</a>

A sequenced collection has first and last elements, allowing support for common operations at either end:

```java
interface SequencedCollection<E> extends Collection<E> {
    // new method
    SequencedCollection<E> reversed();
    // methods promoted from Deque
    void addFirst(E);
    void addLast(E);
    E getFirst();
    E getLast();
    E removeFirst();
    E removeLast();
}
```

A sequenced set is a `Set` that is a `SequencedCollection` that contains no duplicate elements:

```java
interface SequencedSet<E> extends Set<E>, SequencedCollection<E> {
    SequencedSet<E> reversed();    // covariant override
}
```

A sequenced map is a `Map` whose entries have a defined encounter order:

```java
interface SequencedMap<K,V> extends Map<K,V> {
    // new methods
    SequencedMap<K,V> reversed();
    SequencedSet<K> sequencedKeySet();
    SequencedCollection<V> sequencedValues();
    SequencedSet<Entry<K,V>> sequencedEntrySet();
    V putFirst(K, V);
    V putLast(K, V);
    // methods promoted from NavigableMap
    Entry<K, V> firstEntry();
    Entry<K, V> lastEntry();
    Entry<K, V> pollFirstEntry();
    Entry<K, V> pollLastEntry();
}
```

The `Collections` utility class has also been extended to create unmodifiable wrappers for the three new types:

* `Collections.unmodifiableSequencedCollection(sequencedCollection)`
* `Collections.unmodifiableSequencedSet(sequencedSet)`
* `Collections.unmodifiableSequencedMap(sequencedMap)`

These additions to the collections framework ensure that common operations like getting the last element or iterating in reversed order become a lot easier. Moreover, ... TODO 

#### What's Different From Java 20?

Java 20 didn't contain anything related to sequenced collections yet, so everything that you read about it so far is as new as it gets! However, the feature that now has been delivered is the result of an incremental evolution of an earlier proposal called [reversible collections](http://mail.openjdk.org/pipermail/core-libs-dev/2021-April/076461.html). And if you're seriously up for some digging: the earliest form of this proposal was Tagir Valeev's proposal from 2020 called [OrderedMap/OrderedSet](http://mail.openjdk.org/pipermail/core-libs-dev/2020-April/066028.html). These proposals have played a big part in shaping the feature that is now called 'sequenced collections' and that we get to enjoy in Java 21.

#### More Information

For more information on this feature, see [JEP 431](https://openjdk.org/jeps/431).

### JEP 452: Key Encapsulation Mechanism API

A protocol like Transport Layer Security (TLS) relies heavily on public key encryption schemes in order to provide a secure way for a sender and recipient to share information.
But public key encryption schemes are usually less efficient than symmetric encryption schemes, which instead focus on establishing a shared symmetric key as the basis of all future communication between sender and recipient. Such a key is typically produced by a _key encapsulation mechanism_ (or KEM), and JEP 452 proposes to introduce such an API to the JDK. 

#### Components

A KEM needs the following components:

* a key pair generation function that returns a key pair containing a public key and a private key.

> This function is already covered by the existing [`KeyPairGenerator` API](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/security/KeyPairGenerator.html).
 
* a key encapsulation function, called by the sender, that takes the receiver's public key and an encryption option; it returns a secret key and a _ciphertext_. The sender sends the key encapsulation message to the receiver.

> The new `KEM` class contains an encapsulation function.
   
* a key decapsulation function, called by the receiver, that takes the receiver's private key and the received key encapsulation message; it returns the secret key.

> The new `KEM` class contains a decapsulation function.

#### `KEM` class

For illustration purposes, the `KEM` class that is mentioned in the JEP is listed below:

```java
package javax.crypto;

public class DecapsulateException extends GeneralSecurityException;

public final class KEM {

    public static KEM getInstance(String alg)
        throws NoSuchAlgorithmException;
    public static KEM getInstance(String alg, Provider p)
        throws NoSuchAlgorithmException;
    public static KEM getInstance(String alg, String p)
        throws NoSuchAlgorithmException, NoSuchProviderException;

    public static final class Encapsulated {
        public Encapsulated(SecretKey key, byte[] encapsulation, byte[] params);
        public SecretKey key();
        public byte[] encapsulation();
        public byte[] params();
    }

    public static final class Encapsulator {
        String providerName();
        int secretSize();           // Size of the shared secret
        int encapsulationSize();    // Size of the key encapsulation message
        Encapsulated encapsulate();
        Encapsulated encapsulate(int from, int to, String algorithm);
    }

    public Encapsulator newEncapsulator(PublicKey pk)
            throws InvalidKeyException;
    public Encapsulator newEncapsulator(PublicKey pk, SecureRandom sr)
            throws InvalidKeyException;
    public Encapsulator newEncapsulator(PublicKey pk, AlgorithmParameterSpec spec,
                                        SecureRandom sr)
            throws InvalidAlgorithmParameterException, InvalidKeyException;

    public static final class Decapsulator {
        String providerName();
        int secretSize();           // Size of the shared secret
        int encapsulationSize();    // Size of the key encapsulation message
        SecretKey decapsulate(byte[] encapsulation) throws DecapsulateException;
        SecretKey decapsulate(byte[] encapsulation, int from, int to,
                              String algorithm) throws DecapsulateException;
    }

    public Decapsulator newDecapsulator(PrivateKey sk)
            throws InvalidKeyException;
    public Decapsulator newDecapsulator(PrivateKey sk, AlgorithmParameterSpec spec)
            throws InvalidAlgorithmParameterException, InvalidKeyException;
}
```

> The `getInstance` methods create a new `KEM` object that implements the specified algorithm.

And here is an example of how to use this class (again, taken from the JEP):

```java
// Receiver side
KeyPairGenerator g = KeyPairGenerator.getInstance("ABC");
KeyPair kp = g.generateKeyPair();
publishKey(kp.getPublic());

// Sender side
KEM kemS = KEM.getInstance("ABC-KEM");
PublicKey pkR = retrieveKey();
ABCKEMParameterSpec specS = new ABCKEMParameterSpec(...);
KEM.Encapsulator e = kemS.newEncapsulator(pkR, specS, null);
KEM.Encapsulated enc = e.encapsulate();
SecretKey secS = enc.key();
sendBytes(enc.encapsulation());
sendBytes(enc.params());

// Receiver side
byte[] em = receiveBytes();
byte[] params = receiveBytes();
KEM kemR = KEM.getInstance("ABC-KEM");
AlgorithmParameters algParams = AlgorithmParameters.getInstance("ABC-KEM");
algParams.init(params);
ABCKEMParameterSpec specR = algParams.getParameterSpec(ABCKEMParameterSpec.class);
KEM.Decapsulator d = kemR.newDecapsulator(kp.getPrivate(), specR);
SecretKey secR = d.decapsulate(em);

// secS and secR will be identical
```

#### What's Different From Java 20?

The Key Encapsulation Mechanism API didn't exist yet in Java 20; it was newly added in Java 21.

#### More Information

There is a bit more to this API than we were able to cover here (like different _KEM configurations_, for example), so to get the full picture you could have a look at [JEP 452](https://openjdk.org/jeps/452).

## Final thoughts

TODO
