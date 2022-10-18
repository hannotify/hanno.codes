---
layout: post
author: Hanno Embregts
title: "Eleven crazy learnings from the Java 11 certification: wrapper objects - functional interfaces actually CAN contain multiple abstract methods (10/11)"
date: 30-08-2022 19:00:00 +0200
image: /assets/images/blog/lanyard.jpg
tags: 
- java
- certification
---

In the summer of 2021, I got my Java 11 certification. I expected it to be quite a breeze, because I'd been a Java developer for 14 years and surely I should have seen it all by now, right? Turned out I was very wrong. I came across lots of things that I didn't even know were possible with Java. In this weekly blog series I will go through 11 of these 'crazy learnings' that surprised me the most, even as an experienced developer. We're diving into functional interfaces today.

## Learning to use lambda expressions

When I first learned to use lambda expressions, I thought I had Java's functional interfaces all figured out.
Ever since Java 8 was released, interfaces with just a single abstract method are called 'functional interfaces', and you can communicate that intent by annotating the interface with `@FunctionalInterface`.
Sounded easy enough.

## Exam question on functional interfaces

Then I did a Java Certification practice exam and got an exam question that looked like this.
(I changed the interface and method names to make it fit our familiar conference domain)

> Which types in the code listing below are actually functional interfaces? (4 answers required)

```java
@FunctionalInterface
abstract class Venue {
    public abstract void open();
}

@FunctionalInterface
interface Speaker {
    void speak();
}

@FunctionalInterface
interface Badge {
    void print();
    default void flip() {
        System.out.println("Badge has flipped... again!");
    }
}

@FunctionalInterface
interface IgniteSpeaker extends Speaker {
    void stressOutOverTimeLimit(int timeLimit);
}

@FunctionalInterface
interface ByteSizeSpeaker extends Speaker {
    default void speak() {
        System.out.println("Speaking on a great subject in a Byte Size format");
    }
    void stressOutOverTimeLimit(int timeLimit);
}

@FunctionalInterface
interface Room {
    void book(LocalTime timeslot);
    boolean equals(Object otherRoom);
    String toString();
}
```

When I'd finished reading this question, I suddenly wasn't so sure any more that I knew all about functional interfaces! 
"Just a single abstract method" isn't really simple any more when inheritance and overridden methods from the `Object` class are involved.
So let's cycle through the six examples and write up a few traits of a functional interface, shall we?

## Traits of a functional interface

1. Functional interfaces can't be abstract classes - they actually have to be 'interfaces'
(so `Venue` won't even compile because of this).
1. Functional interfaces should contain a single abstract method.
2. Default methods have an implementation, so they're not abstract. This means that a functional interface can contain both a single abstract method and one or more default methods.
3. From trait #2 we can deduce that a functional interface can't contain multiple abstract methods. This includes inherited abstract methods, so `IgniteSpeaker` can't be a functional interface. It adds an abstract method on top of the one it inherited from `Speaker`.
4. However, if an interface overrides an inherited abstract method with a default method implementation, it is no longer abstract. The interface will still be functional if it adds an abstract method of its own.
5. If an interface declares an abstract method overriding one of the public methods of `java.lang.Object`, that does not count toward the interface's abstract method count since any implementation of the interface will have an implementation from `java.lang.Object` or elsewhere.

Based on this list of traits, we come to the conclusion that the following four types are functional interfaces:

* `Speaker`
* `Badge`
* `ByteSizeSpeaker`
* `Room`

Yes, even the `Room` interface is functional, despite its *multiple abstract methods*. Because these methods are overridden from `Object`, they do not count toward the abstract method count. 🤯

## Informational purposes

The final thing to remember is that the `@FunctionalInterface` annotation is meant for informational purposes. The compiler will treat any interface meeting the definition of a functional interface as a functional interface regardless of whether or not the annotation is present on the interface. That being said, when the `@FunctionalInterface` annotation *is* present, the compiler will check the interface for the traits we listed earlier and report a compile error if one of them isn't met. So in that case you can be sure that the interface is actually functional.

![Lanyard](/assets/images/blog/lanyard.jpg)
> Photo by Pavel Danilyuk from <a href="https://www.pexels.com/photo/badges-and-print-outs-on-a-gray-surface-8761744/">Pexels</a>

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
10. **Functional interfaces actually CAN contain multiple abstract methods**
11. [Passing arguments to method references]({% post_url 2022-09-06-eleven-crazy-learnings-passing-arguments-to-method-references %})

