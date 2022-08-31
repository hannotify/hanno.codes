---
layout: post
author: Hanno Embregts
title: "Eleven crazy learnings from the Java 11 certification: division by zero (5/11)"
date: 26-07-2022 18:00:00 +0200
image: /images/blog/maths.jpg
tags: 
- java
- certification
---

In the summer of 2021, I got my Java 11 certification. I expected it to be quite a breeze, because I'd been a Java developer for 14 years and surely I should have seen it all by now, right? Turned out I was very wrong. I came across lots of things that I didn't even know were possible with Java. In this weekly blog series I will go through 11 of these 'crazy learnings' that surprised me the most, even as an experienced developer. This week we'll get mathematical as we focus on dividing by zero in Java.

## Dividing by zero has no meaning

What happens if you divide a number by zero?
You probably learned all about this mathematical quirk in your middle or high school mathematics class.
In ordinary arithmetic, an expression that consists of a division by zero has no meaning, which is why we were taught that such an expression evaluates to *undefined*.
This simple rule is reflected in the Java language by the `java.lang.ArithmeticException`.
An exception of this type is thrown each time you try to divide by zero.
At least, that was my belief until I got my Java 11 certification. 

## Dividing ints

Sure enough, when we would divide any Integer in Java by zero...

```java
public class DivisionByZero {
    public static void main(String[] args) {
        System.out.println(divide(42, 0));
    }

    public static int divide(int dividend, int divisor) {
        return dividend / divisor;
    }
}
```

...we would get an ArithmeticException thrown at runtime:

> `Exception in thread "main" java.lang.ArithmeticException: / by zero`

## Dividing floats and doubles

OK, so we get an ArithmeticException when dividing an integer by zero.
And I knew already that similar behaviour is displayed when dividing `BigDecimal`s.
So when I got a practice exam question on the use case demonstrated above, but with floats instead of ints, I didn't hesitate and marked the answer that said "RuntimeException is thrown".
Turned out I was wrong!

Now, if we would overload the `divide` method for floats and doubles, we would quickly discover why.

```java
public class DivisionByZero {
    public static void main(String[] args) {
        //System.out.println(divide(42, 0));
        System.out.println(divide(42f, 0f));
        System.out.println(divide(42.0, 0.0));
    }

    public static int divide(int dividend, int divisor) {
        return dividend / divisor;
    }

    public static float divide(float dividend, float divisor) {
        return dividend / divisor;
    }

    public static double divide(double dividend, double divisor) {
        return dividend / divisor;
    }
}
```

This program returns:

> `Infinity`
> 
> `Infinity`

So apparently floats and doubles can hold *infinity* values, as opposed to integers and BigDecimals.

If we turn to the [Java Language Specification](https://docs.oracle.com/javase/specs/jls/se11/html/jls-15.html#jls-15.17.2), it turns out I could have known about this behaviour had I taken the time to read the specification through. Because this is what it has to say on the subject:

> Division of a nonzero finite value by a zero results in a signed infinity.

and:

> Division of a zero by a zero results in NaN.

(This behaviour is in accordance with the IEEE Standard for Floating-Point Arithmetic ([IEEE 754](https://en.wikipedia.org/wiki/IEEE_754)).)

![Maths](/images/blog/maths.jpg)
> Image from <a href="https://pxhere.com/nl/photo/464083">PxHere</a>

## Other blog posts in this series

Did you miss a blog post in this series? Here's a list of all posts that have been published so far:

1. [A few freaky array declarations]({% post_url 2022-06-28-eleven-crazy-learnings-initialising-arrays %})
2. [Stream elements should implement Comparable](/2022/07/05/eleven-crazy-learnings-stream-elements-comparable.html)
3. [Accessing static interface methods](/2022/07/12/eleven-crazy-learnings-accessing-static-interface-methods.html)
4. [Anonymous subclasses in enums](/2022/07/19/eleven-crazy-learnings-anonymous-subclass-in-enum.html)
5. **Division by zero**
6. [Method overloading priorities]({% post_url 2022-08-02-eleven-crazy-learnings-method-overloading-priorities %})
7. [The crazy stuff that is allowed in switch statements]({% post_url 2022-08-09-eleven-crazy-learnings-crazy-stuff-in-switch-statements %})
8. [Equality in cloned arrays]({% post_url 2022-08-16-eleven-crazy-learnings-equality-in-cloned-arrays %})
9. [Wrapper objects: some are more equal than others]({% post_url 2022-08-23-eleven-crazy-learnings-wrapper-objects-some-are-more-equal-than-others %})
10. [Functional interfaces actually CAN contain multiple abstract methods]({% post_url 2022-08-30-eleven-crazy-learnings-functional-interfaces %})