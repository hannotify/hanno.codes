---
layout: post
title: "Eleven crazy learnings from the Java 11 certification: stream elements should implement Comparable (2/11)"
date: 05-07-2022 10:35:00 +0200
image: /assets/images/blog/clock.jpg
tags: 
- java
- certification
---

In the summer of 2021, I got my Java 11 certification. I expected it to be quite a breeze, because I'd been a Java developer for 14 years and surely I should have seen it all by now, right? Turned out I was very wrong. I came across lots of things that I didn't even know were possible with Java. In this weekly blog series I will go through 11 of these 'crazy learnings' that surprised me the most, even as an experienced developer. This week we dive into stream elements.

## Stream elements should implement Comparable

Imagine we are building a conference schedule, and to that end we want to produce a list of talks that is sorted by the start times.

Well, when the `Talk` class looks like this...

```java
public class Talk {
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

...we can obtain the sorted talk list by using a stream of `Talk`s and sorting it.

```java
return Stream.of(
        new Talk("Bugs Bunny", "Carrots Are Awesome!", LocalTime.of(11, 0)),
        new Talk("Road Runner", "Stop Living Too Slow", LocalTime.of(9, 30)),
        new Talk("Tweety", "Ban All Cats Off The Internet", LocalTime.of(14, 45))
).sorted().collect(Collectors.toCollection(TreeSet::new));
```

That's the theory, at least. Because when I ran this piece of code for the first time, this is what I got:

```
java.lang.ClassCastException: class com.github.hannotify.elevencrazyjavathings.number10.Talk cannot be cast to class java.lang.Comparable (com.github.hannotify.elevencrazyjavathings.number10.Talk is in unnamed module of loader 'app'; java.lang.Comparable is in module java.base of loader 'bootstrap')
    at java.base/java.util.Comparators$NaturalOrderComparator.compare(Comparators.java:47)
    at java.base/java.util.TimSort.countRunAndMakeAscending(TimSort.java:355)
    at java.base/java.util.TimSort.sort(TimSort.java:220)
    at java.base/java.util.Arrays.sort(Arrays.java:1515)
    (...)
```

It certainly looks like the sorting is not working yet. 
And the error message is quite unusual, as it doesn't seem to have anything to do with sorting in the first place.
The key here is the mention of the `Comparable` interface.
Apparently the `sorted()` method that we're calling on the stream expects its elements to implement `Comparable`.

And sure enough, when we get to the JavaDoc of the method `Stream.sorted()`, this is what we read:

> Returns a stream consisting of the elements of this stream, sorted according to natural order. If the elements of this stream are not Comparable, a java.lang.ClassCastException may be thrown when the terminal operation is executed.

Well, *now* it makes perfect sense!
So to get this example working, we should make sure that `Talk` implements `Comparable`:

```java
class Talk implements Comparable<Talk> {
    private final String speaker;
    private final String title;
    private final LocalTime startTime;

    public Talk(String speaker, String title, LocalTime startTime) {
        this.speaker = speaker;
        this.title = title;
        this.startTime = startTime;
    }

    @Override
    public int compareTo(Talk otherTalk) {
        return startTime.compareTo(otherTalk.startTime) ;
    }
}
```

Alternatively, we can call the overloaded `sorted()` method, which takes a `Comparator`:

```java
return Stream.of(
        new Talk("Bugs Bunny", "Carrots Are Awesome!", LocalTime.of(11, 0)),
        new Talk("Road Runner", "Stop Living Too Slow", LocalTime.of(9, 30)),
        new Talk("Tweety", "Ban All Cats Off The Internet", LocalTime.of(14, 45))
).sorted(Comparator.comparing(Talk::getStartTime)).collect(Collectors.toCollection(TreeSet::new));
```

But for this to work you would need to add a `getStartTime()` method to the `Talk` class, of course.

So this is how you can sort stream elements, even when you're struggling with a few `ClassCastExpections`. Next week we'll take a look at static interface methods!

![Clock](/assets/images/blog/clock.jpg)
> Image from <a href="https://pxhere.com/nl/photo/883658">PxHere</a>

## Other blog posts in this series

Did you miss a blog post in this series? Here's a list of all posts that have been published so far:

1. [A few freaky array declarations](/2022/06/28/eleven-crazy-learnings-initialising-arrays.html)
2. **Stream elements should implement Comparable**
3. [Accessing static interface methods](/2022/07/12/eleven-crazy-learnings-accessing-static-interface-methods.html)
4. [Anonymous subclasses in enums](/2022/07/19/eleven-crazy-learnings-anonymous-subclass-in-enum.html)
5. [Division by zero]({% post_url 2022-07-26-eleven-crazy-learnings-division-by-zero %})
6. [Method overloading priorities]({% post_url 2022-08-02-eleven-crazy-learnings-method-overloading-priorities %})
7. [The crazy stuff that is allowed in switch statements]({% post_url 2022-08-09-eleven-crazy-learnings-crazy-stuff-in-switch-statements %})
8. [Equality in cloned arrays]({% post_url 2022-08-16-eleven-crazy-learnings-equality-in-cloned-arrays %})
9. [Wrapper objects: some are more equal than others]({% post_url 2022-08-23-eleven-crazy-learnings-wrapper-objects-some-are-more-equal-than-others %})
10. [Functional interfaces actually CAN contain multiple abstract methods]({% post_url 2022-08-30-eleven-crazy-learnings-functional-interfaces %})
11. [Passing arguments to method references]({% post_url 2022-09-06-eleven-crazy-learnings-passing-arguments-to-method-references %})
