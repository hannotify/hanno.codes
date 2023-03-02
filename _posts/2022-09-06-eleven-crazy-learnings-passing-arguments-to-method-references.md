---
layout: post
title: "Eleven crazy learnings from the Java 11 certification: passing arguments to method references (11/11)"
date: 06-09-2022 20:00:00 +0200
header:
  teaser: /assets/images/blog/auditorium.jpg
excerpt: For this final part of the blog series we'll dive into method references.
tags: 
- java
- certification
---

In the summer of 2021, I got my Java 11 certification. I expected it to be quite a breeze, because I'd been a Java developer for 14 years and surely I should have seen it all by now, right? Turned out I was very wrong. I came across lots of things that I didn't even know were possible with Java. In this weekly blog series I will go through 11 of these 'crazy learnings' that surprised me the most, even as an experienced developer. The final 'crazy learning' I'll be blogging about has to do with method references.

## Prefering method references over lambdas

Using a method reference instead of a lambda expression can make your code more concise and readable.
Whenever I can, I use a method reference rather than a lambda expression.
And modern IDEs like IntelliJ IDEA will even suggest to turn a lambda into a method reference.

![IntelliJ suggests a method reference](/assets/images/blog/intellij-suggests-method-reference.png)

## Lambda expressions that take parameters

However, when a lambda expression takes a parameter, I used to just leave the lambda there - I didn't really know how to turn it into a method reference. 
Because it almost seemed impossible to let the method reference take a parameter.
Right?!

Well, when I studied for my OCP Java Developer 11 exam, I learned that there actually *is* a way to achieve this.
Let's illustrate it with a code example.

## Modelling a conference venue

Imagine we want to model a conference venue.
Let's say a venue has a name and a certain capacity.
And we want to provide constructors to provide both name and capacity, or just one of them, or even none of them:

```java
public class Venue {
    final String name;
    final int capacity;

    final static String DEFAULT_NAME = "Anonymous venue";
    final static int DEFAULT_CAPACITY = 0;

    Venue() {
        this(DEFAULT_NAME, DEFAULT_CAPACITY);
    }

    Venue(String name) {
        this(name, DEFAULT_CAPACITY);
    }

    Venue(int capacity) {
        this(DEFAULT_NAME, capacity);
    }

    Venue(String name, int capacity) {
        this.name = name;
        this.capacity = capacity;
    }
}
```

## Creating a Venue using a lambda expression

Now, could we create a `Venue` using a lambda expression?
Sure we can!

```java
// 1. call the default constructor
Supplier<Venue> anonymousVenueSupplier = () -> new Venue();

// 2. call the constructor with a String argument
Supplier<Venue> tinyClassroomSupplier = () -> new Venue("Classroom");

// 3. call the constructor with an int argument
Supplier<Venue> largeAnonymousVenueSupplier = () -> new Venue(200);

// 4. call the constructor with both a String and an int argument
Supplier<Venue> regularClassroomSupplier = () -> new Venue("Classroom", 30);
```

## Creating a Venue using a method reference

OK, so we can call all four constructor overloads from within a lambda.
Now, could we also make this work with method references?

```java
// 1. call the default constructor
Supplier<Venue> anonymousVenueSupplier = Venue::new;

// 2. how can I pass a value to a method reference? 🤔
Supplier<Venue> tinyClassroomSupplier = Venue::new("Classroom"); // doesn't compile!
```

Obviously what we tried at #2 doesn't work.
To get to the solution we have to think about what a method reference actually tries to accomplish.

The method reference at #1 gets the reference of the `Venue` constructor that does not take any argument. This is also indicated by the type of the variable `tinyClassroomSupplier` - which is `Supplier<Venue>`. As you may recall, the functional interface `Supplier` serves as an interface for lambdas that expect no parameters, but produce a return value. So what if we would try a different functional interface here? (one that *can* take a parameter)

```java
// 2. call the constructor with a String argument
Function<String, Venue> tinyClassroomFunction = Venue::new;
tinyClassroomFunction.apply("Classroom");
```

Yay, we have valid code on our hands! 🥳
Because our method reference is a `Function<String, Venue>` here, it will get the reference of the `Venue` constructor that takes a single `String` argument.
And we can provide a value for `Venue.name` by passing it to an invocation of `Function.apply`.

We can call the constructor overload that takes an int using the same way:

```java
// 3. call the constructor with an int argument
Function<Integer, Venue> largeAnonymousVenueFunction = Venue::new;
largeAnonymousVenueFunction.apply(200);
```

This works because of [autoboxing](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html) - alternatively we could have used an `IntFunction<Venue>` which will work with `int` primitives directly.

## Calling a constructor with two arguments 

Now if a `Function` enables us to call a method or constructor that expect a single argument, what would we need when we want to provide two arguments?
In such a case we can use a `BiFunction`, like so:

```java
BiFunction<String, Integer, Venue> regularClassroomBiFunction = Venue::new;
regularClassroomBiFunction.apply("Classroom", 30);
```

So by assigning a method reference to a functional interface type that takes a parameter (like `Function` as we've seen, but `Consumer` can also work - when no return type is required), it is possible to pass arguments to it.
And I never knew about this, until I came across it during my preparations for the OCP Java Developer exam.
But I'm really glad I did!

![Lanyard](/assets/images/blog/auditorium.jpg)
> Photo by Pavel Danilyuk from <a href="https://www.pexels.com/photo/badges-and-print-outs-on-a-gray-surface-8761744/">Pexels</a>

## Conclusion of this series

This was the final chapter of my "Eleven Crazy Learnings from the Java 11 Certification" blog series.
I hope you've enjoyed reading it.
If you should decide to get Java Certified yourself, I do recommend you write down the things you didn't know yet - just like I did.
It will help you remember your own 'crazy learnings' a bit better.

It will probably take me a bit longer than a week to publish the next blog post (the weekly schedule that went on for 11 weeks has taken its toll on me 🙂), but don't worry - I'll continue posting regularly about all things Java, software development and public speaking.
Until next time! 👋

## Other blog posts in this series

Did you miss a blog post in this series? Here's a list of all posts that have been published so far:

1. [A few freaky array declarations]({% post_url 2022-06-28-eleven-crazy-learnings-initialising-arrays %})
2. [Stream elements should implement Comparable](/2022/07/05/eleven-crazy-learnings-stream-elements-comparable.html)
3. [Accessing static interface methods](/2022/07/12/eleven-crazy-learnings-accessing-static-interface-methods.html)
4. [Anonymous subclasses in enums](/2022/07/19/eleven-crazy-learnings-anonymous-subclass-in-enum.html)
5. [Division by zero]({% post_url 2022-07-26-eleven-crazy-learnings-division-by-zero %})
6. [Method overloading priorities]({% post_url 2022-08-02-eleven-crazy-learnings-method-overloading-priorities %})
7. [The crazy stuff that is allowed in switch statements]({% post_url 2022-08-09-eleven-crazy-learnings-crazy-stuff-in-switch-statements %})
8. [Equality in cloned arrays]({% post_url 2022-08-16-eleven-crazy-learnings-equality-in-cloned-arrays %})
9. [Wrapper objects: some are more equal than others]({% post_url 2022-08-23-eleven-crazy-learnings-wrapper-objects-some-are-more-equal-than-others %})
10. [Functional interfaces actually CAN contain multiple abstract methods]({% post_url 2022-08-30-eleven-crazy-learnings-functional-interfaces %})
11. **Passing arguments to method references**
