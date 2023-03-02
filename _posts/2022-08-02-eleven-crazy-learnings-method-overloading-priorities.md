---
layout: post
title: "Eleven crazy learnings from the Java 11 certification: method overloading priorities (6/11)"
date: 02-08-2022 20:00:00 +0200
header:
  teaser: /assets/images/blog/scrabble.jpg
tags: 
- java
- certification
---

In the summer of 2021, I got my Java 11 certification. I expected it to be quite a breeze, because I'd been a Java developer for 14 years and surely I should have seen it all by now, right? Turned out I was very wrong. I came across lots of things that I didn't even know were possible with Java. In this weekly blog series I will go through 11 of these 'crazy learnings' that surprised me the most, even as an experienced developer. This week's blog is about about method overloading priorities.

## Remembering priorities

Questions about method overloading priorities form a notorious part of any Java certification, regardless of the version. 
So of course I should have been prepared when the first practice exam I did immediately presented me with such a question.
My strategy for these questions always used to be: just learn the priorities by heart.
But of course, for this first practice exam, I didn't remember the exact order and I got the question wrong.

The code that was part of the practice question looked somewhat like this:

```java
public class MethodOverloadingPriorities {

    public static void main(String[] args) {
        printSum(2, 7);
    }

    public static void printSum(Integer a, Integer b){
        System.out.format("In printSum(Integer): %d%n", a + b);
    }

    public static void printSum(double a, double b){
        System.out.format("In printSum(double): %f%n", a + b);
    }

    public static void printSum(int... ints){
        System.out.format("In printSum(int...): %d%n", Arrays.stream(ints).sum());
    }
}
```

You might be tempted to think that the varargs overload would be preferred, because we're passing int literals in the main method.
But Java actually prioritises *widening* over everything else, so the double overload is executed.
If the double overload wouldn't be present in the source file, *boxing* would be preferred over varargs and the Integer overload would be used.

So these were the method overloading priorities that I set out to learn by heart:

1. widening
2. boxing
3. varargs

## Remembering the rationale instead

When I talked to my coworker Tom about this (you should [follow him](https://twitter.com/TCoolsIT) by the way!), he had an interesting view on the matter.
He pointed out that the order of preference wasn't arbitrarily chosen, but had something to do with backwards compatibility.
And sure enough, when I looked it up in the Java language specification, I came across the following part:

> The process of determining applicability begins by determining the potentially applicable methods (§15.12.2.1). Then, **to ensure compatibility with the Java programming language prior to Java SE 5.0**, the process continues in three phases:
>
> 1. The first phase performs overload resolution without permitting boxing or unboxing conversion, or the use of variable arity method invocation. If no applicable method is found during this phase then processing continues to the second phase.
> 
> **This guarantees that any calls that were valid in the Java programming language before Java SE 5.0 are not considered ambiguous as the result of the introduction of variable arity methods, implicit boxing and/or unboxing.** However, the declaration of a variable arity method (§8.4.1) can change the method chosen for a given method method invocation expression, because a variable arity method is treated as a fixed arity method in the first phase. For example, declaring m(Object...) in a class which already declares m(Object) causes m(Object) to no longer be chosen for some invocation expressions (such as m(null)), as m(Object[]) is more specific.
>
> 1. The second phase performs overload resolution while allowing boxing and unboxing, but still precludes the use of variable arity method invocation. If no applicable method is found during this phase then processing continues to the third phase.
>
> This ensures that a method is never chosen through variable arity method invocation if it is applicable through fixed arity method invocation.
>
> 1. The third phase allows overloading to be combined with variable arity methods, boxing, and unboxing.

(from the [Java 11 Language Specification, paragraph 15.12.2](https://docs.oracle.com/javase/specs/jls/se11/html/jls-15.html#jls-15.12.2))

To summarize: the older feature is always preferred over the newer feature to ensure backwards compatibility.

This meant that I didn't have to learn some boring list by heart; I just had to assess for each possible method overload how long it has been possible in Java to execute that specific overload and work out the right priorities from there.

This is why widening is preferred over boxing and varargs, because it's an 'older' language feature.

## What if the 'oldest' overloading rule doesn't work?

> Many thanks to [Tom Cools](https://twitter.com/TCoolsIT) & [Jan-Hendrik Kuperus](https://twitter.com/jhkuperus) for providing me with [this addendum via Twitter](https://twitter.com/jhkuperus/status/1554756615953342464)!

Now varargs and autoboxing are both Java 5 features, so the 'oldest' overloading rule doesn't work to distinguish their priorities.
Between those two, the most performant wins, which is autoboxing. 
Because varargs create arrays, and that is heavier work than just (un)boxing.

Also, variable arity is usually considered last to prevent confusion. 
Which is also why the varargs-param must be the last one.

## General advice

I think this point brings us to a useful piece of general advice.
Whenever you're considering to learn something by heart that's about a programming language's behaviour, first spend some time to understand the rationale behind it.
Chances are you don't have to learn anything by heart at all and you'll end up with a better understanding of the programming language instead.
Which is arguably the far better option.

![Scrabble](/assets/images/blog/scrabble.jpg)
> Photo by Brett Jordan from <a href="https://www.pexels.com/photo/close-up-shot-of-scrabble-tiles-8241829">Pexels</a>

## Other blog posts in this series

Did you miss a blog post in this series? Here's a list of all posts that have been published so far:

1. [A few freaky array declarations]({% post_url 2022-06-28-eleven-crazy-learnings-initialising-arrays %})
2. [Stream elements should implement Comparable](/2022/07/05/eleven-crazy-learnings-stream-elements-comparable.html)
3. [Accessing static interface methods](/2022/07/12/eleven-crazy-learnings-accessing-static-interface-methods.html)
4. [Anonymous subclasses in enums](/2022/07/19/eleven-crazy-learnings-anonymous-subclass-in-enum.html)
5. [Division by zero]({% post_url 2022-07-26-eleven-crazy-learnings-division-by-zero %})
6. **Method overloading priorities**
7. [The crazy stuff that is allowed in switch statements]({% post_url 2022-08-09-eleven-crazy-learnings-crazy-stuff-in-switch-statements %})
8. [Equality in cloned arrays]({% post_url 2022-08-16-eleven-crazy-learnings-equality-in-cloned-arrays %})
9. [Wrapper objects: some are more equal than others]({% post_url 2022-08-23-eleven-crazy-learnings-wrapper-objects-some-are-more-equal-than-others %})
10. [Functional interfaces actually CAN contain multiple abstract methods]({% post_url 2022-08-30-eleven-crazy-learnings-functional-interfaces %})
11. [Passing arguments to method references]({% post_url 2022-09-06-eleven-crazy-learnings-passing-arguments-to-method-references %})