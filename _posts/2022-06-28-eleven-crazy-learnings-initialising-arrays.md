---
layout: post
author: Hanno Embregts
title: "Eleven crazy learnings from the Java 11 certification: a few freaky array declarations (1/11)"
date: 28-06-2022 21:44:00 +0200
tags: 
- java
- certification
---
In the summer of 2021, I got my Java 11 certification. I expected it to be quite a breeze, because I'd been a Java developer for 14 years and surely I should have seen it all by now, right? Turned out I was very wrong. I came across lots of things that I didn't even know were possible with Java. In this weekly blog series I will go through 11 of these 'crazy learnings' that surprised me the most, even as an experienced developer. We start off with a few freaky array declarations.

## Replacing array declarations by 'var'

I really like [local-varable type inference](https://openjdk.org/jeps/286), so when the opportunity arises to replace an explicit local-variable type by `var` I feel really happy. 
When you do this for arrays, however, there is a small caveat.

You see, when I came across this line of code...

```java
int[] elements = new int[] { 0, 1, 2, 3, 5, 8, 13, 21 };
```

...I just replaced `int` by `var` without thinking and thought I was done.

```java
var[] elements = new int[] { 0, 1, 2, 3, 5, 8, 13, 21 }; // doesn't compile!
```

Because we're dealing with an int array here, the square brackets are *part of the data type*, and so `var` needs to replace them also. Which will result in:

```java
var elements = new int[] { 0, 1, 2, 3, 5, 8, 13, 21 }; // much better
```

It's not a very complicated thing to remember, but it'll surely save you some time if you do.

## C-style arrays and multiple declarations on a single line

Java supports two styles of array declaration.

```java
int[] dailyHighscores; // style 1
int allTimeHighscores[]; // style 2
```

The second style is called 'C-style' array declaration and it was added to help C programmers get used to Java. Kinda hard to imagine in a world where 18 Java versions have been released to this day, but the story seems to check out. Defining an array in C indeed means [putting the brackets after the variable name](https://www.w3schools.com/c/c_arrays.php), not before.

The style of array declaration used isn't really relevant as long as we exclusively write single declarations on a single line of code only. Things get a little hairier when we start declaring multiple arrays on a single line of code.

```java
int dailyHighscores[], allTimeHighscores; // second array is an int!
```

In this example, `dailyHighscores` is an int array, but `allTimeHighscores` is an int primitive!
If you want both variables to be int arrays in a single declaration, you'd have to stick to the conventional style of array declaration:

```java
int[] dailyHighscores, allTimeHighscores; // both variables are arrays now
```

To drive the point home, the following declaration...

```java
int[] dailyHighscores, allTimeHighscores, highscoreMatrix[];
```

...would be equivalent to:

```java
int[] dailyHighscores;
int[] allTimeHighscores;
int[][] highscoreMatrix;
```

So there you have it: a few freaky ways to declare arrays. Next week we'll take a look at stream elements!

## Other blog posts in this series

Did you miss a blog post in this series? Here's a list of all posts that have been published so far:

1. A few freaky array declarations
2. [Stream elements should implement Comparable](/2022/07/05/eleven-crazy-learnings-stream-elements-comparable.html)
3. [Accessing static interface methods](2022/07/12/eleven-crazy-learning-accessing-static-interface-methods.html)


![Pinball machine](/images/blog/pinball.jpg)
> Image from <a href="https://pxhere.com/nl/photo/201809>PxHere</a>
