---
layout: post
author: Hanno Embregts
title: "Eleven crazy learnings from the Java 11 certification: wrapper objects - some are more equal than others (9/11)"
date: 23-08-2022 17:00:00 +0200
image: /assets/images/blog/maths-confusion.jpg
tags: 
- java
- certification
---

In the summer of 2021, I got my Java 11 certification. I expected it to be quite a breeze, because I'd been a Java developer for 14 years and surely I should have seen it all by now, right? Turned out I was very wrong. I came across lots of things that I didn't even know were possible with Java. In this weekly blog series I will go through 11 of these 'crazy learnings' that surprised me the most, even as an experienced developer. Today is about equality when you're dealing with wrapper objects.

## Wrapper objects are immutable

When we compare primitives, we are used to using the `==` operator to test equality.
However, when we start using wrapper objects instead of primitives, we have to switch to using invocations of the `equals()` method to test equality.
The reason for this is that all wrapper objects are immutable.
    
For example, when you run the following code...

```java
Integer i = 200;
i++;
```

...what actually happens is something like this:

```java
Integer i = 200;
i = Integer.valueOf(i.intValue() + 1);
```

As you can see, a different Integer object is assigned back to `i`, because of immutability.

## Different identity

Because the wrapper objects are immutable, two separate objects containing the same value will not be equal to each other (at least not according to the `==` operator).
So although two Integer objects might be similar because of the values they hold, they'll still be different based on their *identity*.

Let's try this out using a code example. We've defined a static method that applies the `==` operator to some Integer parameters...

```java
public static boolean areIntegersEqual(Integer a, Integer b) {
    return a == b;
}
```

...and we'll run the following test method to assert that two similar Integers are not equal at all:

```java
@Test
@DisplayName("integerEquals() should return false when the same arguments (200) are passed")
void integerEqualsShouldReturnFalseWith200() {
    assertThat(areIntegersEqual(200, 200)).isFalse();
}
```

## Wrapper object caching

Well, if two Integers that hold the value `200` are not equal to each other, then *surely* the same will be the case when we compare two Integers with the value `10`, right?

```java
@Test
@DisplayName("integerEquals() should return false when the same arguments (10) are passed")
void integerEqualsShouldReturnFalseWith10() {
    assertThat(areIntegersEqual(10, 10)).isFalse();
}
```

```
org.opentest4j.AssertionFailedError: 
Expecting value to be false but was true
Expected :false
Actual   :true
```

How peculiar! There must be a good reason for this behaviour.
Well, of course there is - there always is!

It turns out that, to save on memory, Java 'reuses' all the wrapper objects whose values fall in the following ranges:

* All Boolean values (`true` and `false`)
* All Byte values
* All Character values from `\u0000` to `\u007f` (i.e. `0` to `127` in decimal)
* All Short and Integer values from `-128` to `127`

This means that the `==` operator will always return `true` when their primitive values are the same and belong to the above list of values.

## Java Language Specification

This is what the Java Language Specification has to say on the subject:

> If the value p being boxed is the result of evaluating a constant expression (§15.28) of type boolean, char, short, int, or long, and the result is true, false, a character in the range '\u0000' to '\u007f' inclusive, or an integer in the range -128 to 127 inclusive, then let a and b be the results of any two boxing conversions of p. It is always the case that a == b.

(from the [Java 11 Language Specification, paragraph 5.1.7](https://docs.oracle.com/javase/specs/jls/se11/html/jls-5.html#jls-5.1.7))

In conclusion: some wrapper objects are more equal than others!
And now you also know why and when.

![Confused about maths](/assets/images/blog/maths-confusion.jpg)
> Photo by Karolina Grabowska from <a href="https://www.pexels.com/photo/boy-standing-in-front-of-a-blackboard-with-an-equation-written-on-it-6256110/">Pexels</a>

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
9. **Wrapper objects: some are more equal than others**
10. [Functional interfaces actually CAN contain multiple abstract methods]({% post_url 2022-08-30-eleven-crazy-learnings-functional-interfaces %})
11. [Passing arguments to method references]({% post_url 2022-09-06-eleven-crazy-learnings-passing-arguments-to-method-references %})