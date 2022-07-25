---
layout: post
author: Hanno Embregts
title: "Eleven crazy learnings from the Java 11 certification: accessing static interface methods (3/11)"
date: 12-07-2022 10:35:00 +0200
image: /images/blog/clock.jpg
tags: 
- java
- certification
---

In the summer of 2021, I got my Java 11 certification. I expected it to be quite a breeze, because I'd been a Java developer for 14 years and surely I should have seen it all by now, right? Turned out I was very wrong. I came across lots of things that I didn't even know were possible with Java. In this weekly blog series I will go through 11 of these 'crazy learnings' that surprised me the most, even as an experienced developer. This week is about accessing static interface methods.

## Accessing static interface fields

So: interfaces can have fields, right? They are [always implicitly `public`, `static` and `final`](https://docs.oracle.com/javase/specs/jls/se18/html/jls-9.html#jls-9.3). And multiple ways exist to access them. We'll use the conference schedule example from [last week's post](/2022/07/05/eleven-crazy-learnings-stream-elements-comparable.html) to demonstrate this. 

So imagine we want to store the default length of a given talk. Let's define a `Slot` interface that stores this value.

```java
interface Slot {
    int LENGTH_IN_MINUTES = 50;
}
```

Now, if we let our `Talk` class implement the `Slot` inferface, like so...

```java
class Talk implements Slot {
    private final String speaker;
    private final String title;
    private final LocalTime startTime;

    public Talk(String speaker, String title, LocalTime startTime) {
        this.speaker = speaker;
        this.title = title;
        this.startTime = startTime;
    }
}
```

...we can access the `LENGTH_IN_MINUTES` static interface field in three different ways. The following test method uses JUnit 5 and the AssertJ library to illustrate them:

```java
@Test
void accessingStaticInterfaceField() {
    var carrotsAreAwesome = new Talk("Bugs Bunny", "Carrots Are Awesome!", LocalTime.of(11, 0));

    // #1: access through Talk instance
    assertThat(carrotsAreAwesome.LENGTH_IN_MINUTES).isEqualTo(50);
    
    // #2: access through Talk class
    assertThat(Talk.LENGTH_IN_MINUTES).isEqualTo(50);
    
    // #3: access through Slot interface
    assertThat(Slot.LENGTH_IN_MINUTES).isEqualTo(50);
}
```

## Accessing static interface methods: a lot harder than you might think!

OK, so now that we've learned a few ways to access interface fields, let's move on to interface methods. Let's add a static method to our `Slot` interface:

```java
interface Slot {
    int LENGTH_IN_MINUTES = 50;

    static String lengthDescription() {
        return String.format("This slot lasts for %d minutes.", LENGTH_IN_MINUTES);
    }
}
```

Now, it would make a lot of sense if we were able to access this method in the same three ways as before, right? 

```java
@Test
void accessingStaticInterfaceMethod() {
    var stopLivingTooSlow = new Talk("Road Runner", "Stop Living Too Slow", LocalTime.of(9, 30));

    // #1: access through Talk instance: doesn't compile! 💥
    assertThat(stopLivingTooSlow.lengthDescription()).isEqualTo("This slot lasts for 50 minutes.");

    // #2: access through Talk class: doesn't compile! 💥
    assertThat(Talk.lengthDescription()).isEqualTo("This slot lasts for 50 minutes.");

    // #3: access through Slot interface - compiles OK ✓
    assertThat(Slot.lengthDescription()).isEqualTo("This slot lasts for 50 minutes.");
}
```

So the code for #1 and #2 doesn't compile; the static method can't be called on a `Talk` instance or through the `Talk` class: 

```
java: cannot find symbol
  symbol:   method lengthDescription()
  location: variable stopLivingTooSlow of type com.github.hannotify.elevencrazyjavathings.number9.Talk
```

The only way to call the static interface method is by explicitly calling it through the `Slot` interface, like we did in the code example at #3.

## Rationale

According to the Java Language Specification, [a class does not inherit (...) static methods from its superinterfaces](https://docs.oracle.com/javase/specs/jls/se11/html/jls-8.html#jls-8.4.8). And though this behaviour may seem counter-intuitive at first glance, there's actually a good reason for it. 

Recall that Java allows a class to implement multiple interfaces. What if those interfaces would all define the same static method? What method would then be available to the implementing class? The compiler wouldn't know which one to invoke! So that's why the restriction was put in place, and why developers should explicitly name the interface that contains the static method you want to call.

## Other blog posts in this series

Did you miss a blog post in this series? Here's a list of all posts that have been published so far:

1. [A few freaky array declarations](/2022/06/28/eleven-crazy-learnings-initialising-arrays.html)
2. [Stream elements should implement Comparable](/2022/07/05/eleven-crazy-learnings-stream-elements-comparable.html)
3. Accessing static interface methods (you've just finished reading it!)
4. [Anonymous subclasses in enums](/2022/07/19/eleven-crazy-learnings-anonymous-subclass-in-enum.html)
5. [Division by zero]({% post_url 2022-07-26-eleven-crazy-learnings-division-by-zero %})

![Clock](/images/blog/clock.jpg)
> Image from <a href="https://pxhere.com/nl/photo/883658">PxHere</a>
