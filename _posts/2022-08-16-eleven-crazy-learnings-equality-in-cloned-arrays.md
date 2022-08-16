---
layout: post
author: Hanno Embregts
title: "Eleven crazy learnings from the Java 11 certification: equality in cloned arrays (8/11)"
date: 16-08-2022 17:00:00 +0200
image: /images/blog/auditorium.jpg
tags: 
- java
- certification
---

In the summer of 2021, I got my Java 11 certification. I expected it to be quite a breeze, because I'd been a Java developer for 14 years and surely I should have seen it all by now, right? Turned out I was very wrong. I came across lots of things that I didn't even know were possible with Java. In this weekly blog series I will go through 11 of these 'crazy learnings' that surprised me the most, even as an experienced developer. We'll dive into array equality today.

## Array equality

When are two arrays in Java equal to each other?
Does using the `==` operator yield different results than an invocation of the `equals()` method?
What's up with the `Arrays.equals()` method?
Does the behaviour change when the second array is a clone of the first one?

I can guarantee that you will ask yourself these questions at some point during your preparation for a Java certification.
The answers to these questions differ subtly and in cases like that a good approach is to try out the different variations and see if you can explain the results to yourself.

## An array of Talks

So let's try a few variations by returning to the code example we used in previous installments of the 'Eleven crazy learnings' blog series. 
As you may recall, this example is about conferences, speakers and talks.
Assume that we now want to build up a conference schedule of talks that were selected. 
A rudimentary implementation of that requirement might use arrays of talks for that:

```java
public class ConferenceSchedule {
    private Map<Year, Talk[]> schedule;
    
    public ConferenceSchedule() {
        select(Year.of(2022), new Talk[]{
                new Talk("Bugs Bunny", "Carrots Are Awesome!", LocalTime.of(11, 0)),
                new Talk("Road Runner", "Stop Living Too Slow", LocalTime.of(9, 30)),
                new Talk("Tweety", "Ban All Cats Off The Internet", LocalTime.of(14, 45))});
    }

    public void select(Year year, Talk[] talks) {
        schedule.put(year, talks);
    }

    public Talk[] retrieve(Year year) {
        return schedule.get(year);
    }
}
```

Yes, we could have used a Collection here (or a value object for that matter). 
This blog is about arrays though, so our hands are tied I'm afraid!

## Another array of Talks

Let's extend the constructor of `ConferenceSchedule` and create a second array that contains the exact same data, but is a different instance.

(apparently the 2022 edition of the conference was a smashing success, so they just reprised the schedule 😝)

```java
public ConferenceSchedule() {
        var talks = new Talk[]{
                new Talk("Bugs Bunny", "Carrots Are Awesome!", LocalTime.of(11, 0)),
                new Talk("Road Runner", "Stop Living Too Slow", LocalTime.of(9, 30)),
                new Talk("Tweety", "Ban All Cats Off The Internet", LocalTime.of(14, 45))};
        
        select(Year.of(2022), talks);
        select(Year.of(2023), talks.clone());
    }
```

Consequently we can call the `retrieve()` method with different values for the `Year` parameter to get to these two arrays to compare them.

## Three ways to test array equality

So let's look at three ways to test the equality of these two arrays, using JUnit 5 and AssertJ.
Here's our basic test class:

```java
class TestConferenceSchedule {
    private final ConferenceSchedule schedule = new ConferenceSchedule();

    private Talk[] talks2022;
    private Talk[] talks2023;

    @BeforeEach
    void setup() {
        talks2022 = schedule.retrieve(Year.of(2022));
        talks2023 = schedule.retrieve(Year.of(2023));
    }
}
```

### 1. Using the `==` operator

The `==` operator tests if the two objects compared are the exact same instance.
In our case (because `talks2023` is a cloned version of `talks2022`) they are separate instances, so we expect `false` as a result here:

```java
    @Test
    void testEqualsOperator() {
        assertThat(talks2022 == talks2023).isFalse();
    }
```

### 2. Invoking the `equals()` method

Invoking the equals() method on two arrays with the same contents returns `false` again:

```java
    @Test
    void testEqualsMethodInvocation() {
        assertThat(talks2022.equals(talks2023)).isFalse();
    }
```

This one surprised me at first!
We're all so used to `equals()` implementations that compare objects by value and not by identity (like with Strings, for example).
But arrays don't have a specific overload of the Object.equals() method, so when you call that method on an array, the Object implementation is used. 
Which yields the exact same behaviour as the `==` operator.

### 3. Invoking `Arrays.equals()`

The method `Arrays.equals()` is meant for comparing two arrays by value.
So it makes sense that it would return `true` for our example arrays.
There's one small caveat: you have to make sure that you've overridden the `equals()` method of the type your array is constructed of (in our case the `Talk` class) and provide a meaningful implementation.
Otherwise the `equals()` implementation of Object is used again.

(and I don't know about you, but I've growen a bit tired of that implementation 🙂)

```java
    @Test
    void testArraysEquals() {
        assertThat(Arrays.equals(talks2022, talks2023)).isTrue();
    }
```

> ⚠️ If you want to compare multi-dimensional arrays, you should use `Arrays.deepEquals()` instead. It will also compare nested arrays.

## Cloned arrays

So we've seen that a cloned array is a different instance from the original one, but the items in the array are the same.
And when I say 'the same', I actually mean 'one and the same'.

```java
    @Test
    void testArrayItemsAreTheExactSameInstance() {
        for (int i = 0; i < talks2022.length; i++) {
            assertThat(talks2022[i] == talks2023[i]).isTrue();
        }
    }
```

The test method above actually succeeds!
This is because `clone()` creates a *shallow copy* of the original array.
Meaning that the cloned array is a different instance than the original, but the items in it are the exact same instances than those that were in the original.

So if you remember these things about arrays, equality and cloning of arrays, you'll be more than ready for certification questions on the subject!

![Auditorium](/images/blog/auditorium.jpg)
> Photo from <a href="https://pxhere.com/nl/photo/844533">PxHere</a>

## Other blog posts in this series

Did you miss a blog post in this series? Here's a list of all posts that have been published so far:

1. [A few freaky array declarations]({% post_url 2022-06-28-eleven-crazy-learnings-initialising-arrays %})
2. [Stream elements should implement Comparable](/2022/07/05/eleven-crazy-learnings-stream-elements-comparable.html)
3. [Accessing static interface methods](/2022/07/12/eleven-crazy-learnings-accessing-static-interface-methods.html)
4. [Anonymous subclasses in enums](/2022/07/19/eleven-crazy-learnings-anonymous-subclass-in-enum.html)
5. [Division by zero]({% post_url 2022-07-26-eleven-crazy-learnings-division-by-zero %})
6. [Method overloading priorities]({% post_url 2022-08-02-eleven-crazy-learnings-method-overloading-priorities %})
7. [The crazy stuff that is allowed in switch statements]({% post_url 2022-08-09-eleven-crazy-learnings-crazy-stuff-in-switch-statements %})
8. Equality in cloned arrays (you've just finished reading it!)
