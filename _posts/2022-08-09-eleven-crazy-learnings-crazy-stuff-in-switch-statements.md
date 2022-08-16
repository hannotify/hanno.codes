---
layout: post
author: Hanno Embregts
title: "Eleven crazy learnings from the Java 11 certification: the crazy stuff that is allowed in switch statements (7/11)"
date: 09-08-2022 17:00:00 +0200
image: /images/blog/rating.jpg
tags: 
- java
- certification
---

In the summer of 2021, I got my Java 11 certification. I expected it to be quite a breeze, because I'd been a Java developer for 14 years and surely I should have seen it all by now, right? Turned out I was very wrong. I came across lots of things that I didn't even know were possible with Java. In this weekly blog series I will go through 11 of these 'crazy learnings' that surprised me the most, even as an experienced developer. Today we'll look at the crazy stuff that is allowed in switch statements.

## Valid data types

What data types are valid to apply a switch statement to?
Starting from Java 17, the answer to this question became a bit more complicated with the introduction of the excellent ['Pattern matching for switch'](https://openjdk.org/jeps/406) feature.
But in 2021 I was studying for the Java 11 certification, and in that version the answer to the question is quite straight-forward:

* char, byte, short, int
* Character, Byte, Short, Integer
* String
* enum types

(more info in the [Java Language Specification, chapter 14.11](https://docs.oracle.com/javase/specs/jls/se11/html/jls-14.html#jls-14.11))

## Practice exam question

So armed with this knowledge I did another practice exam, and one of the questions had a code example that looked a lot like the following:

```java
public static void main(String[] args) {
    System.out.println(talkRatingDescription('g'));
}

public static String talkRatingDescription(char rating) {
    var description = "";

    switch (rating) {
        case 'a':
            description = "Great talk!";
            break;
        case 'b':
            description = "Good talk.";
            break;
        case 'c' | 'd':
            description = "Average talk.";
            break;
        default:
            description = "Bad talk.";
    }

    return description;
}
```

And the question was: "what is the output of the main method?"
Well, switching over `char` is perfectly valid, of course, but the `case 'c' | 'd':` immediately caught my attention.
That looked pretty fishy to me. 
In fact, I was quite sure I hadn't seen that construct in a switch statement before.
So I picked the answer option that said "Compilation error".

I was wrong, of course.
The right answer turned out to be:  compilation succeeds and the output is "Average talk".
Huh?!
If compilation actually succeeds, then I would have expected the `default` branch to execute, and that would have resulted in "Bad talk" as output.
Surely I was missing something here.

## Constant expressions

And indeed I was.
It turned out that fishy `case 'c' | 'd':` wasn't just some make-believe 'case or' operator sham.
Because it actually contains a *bitwise operator*, and this makes `'c' | 'd'` an expression.
A [constant expression](https://docs.oracle.com/javase/specs/jls/se11/html/jls-15.html#jls-ConstantExpression) to be more precise, and this kind of expression is actually *allowed* as a case label.

So what does this constant expression evaluate to?
Well, we know that the bitwise or operation only results in zero when both operands are zero.
And if one or both of the operands are one, the result of the bitwise or is also one.
If we now take the binary representations of the chars `c` and `d`, like this...

```
'c' = 99  = 1100011
'd' = 100 = 1100100
```

...then `'c' | 'd'` evaluates to `1100111`, which is equal to 103, or `g` if converted back to a char.
Which means we now know the reason why the method call `talkRatingDescription('g')` returned "Average talk" in the first place!
And I'm quite sure I'll never forget what constant expressions are and that switch labels are quite happy if you feed them one.

![Rating](/images/blog/rating.jpg)
> Photo by Mohamed Hassan from <a href="https://pxhere.com/nl/photo/1451207">PxHere</a>

## Other blog posts in this series

Did you miss a blog post in this series? Here's a list of all posts that have been published so far:

1. [A few freaky array declarations]({% post_url 2022-06-28-eleven-crazy-learnings-initialising-arrays %})
2. [Stream elements should implement Comparable](/2022/07/05/eleven-crazy-learnings-stream-elements-comparable.html)
3. [Accessing static interface methods](/2022/07/12/eleven-crazy-learnings-accessing-static-interface-methods.html)
4. [Anonymous subclasses in enums](/2022/07/19/eleven-crazy-learnings-anonymous-subclass-in-enum.html)
5. [Division by zero]({% post_url 2022-07-26-eleven-crazy-learnings-division-by-zero %})
6. [Method overloading priorities]({% post_url 2022-08-02-eleven-crazy-learnings-method-overloading-priorities %})
7. The crazy stuff that is allowed in switch statements (you've just finished reading it!)
8. [Equality in cloned arrays]({% post_url 2022-08-16-eleven-crazy-learnings-equality-in-cloned-arrays %})

