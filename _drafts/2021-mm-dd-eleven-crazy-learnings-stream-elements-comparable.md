---
layout: post
author: Hanno Embregts
title: "Eleven crazy learnings from the Java 11 certification: stream elements should implement Comparable (2/11)"
#date: 29-10-2021 15:31:21 +0200
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

## Other blog posts in this series

Did you miss a blog post in this series? Here's a list of all posts that have been published so far:

1. [A few freaky array declarations](/2022/06/28/eleven-crazy-learnings-initialising-arrays.html)
2. Stream elements should implement Comparable  


