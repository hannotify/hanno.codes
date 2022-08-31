---
layout: post
author: Hanno Embregts
title: "Eleven crazy learnings from the Java 11 certification: anonymous subclasses in enums (4/11)"
date: 19-07-2022 19:35:00 +0200
image: /images/blog/conference.jpg
tags: 
- java
- certification
---

In the summer of 2021, I got my Java 11 certification. I expected it to be quite a breeze, because I'd been a Java developer for 14 years and surely I should have seen it all by now, right? Turned out I was very wrong. I came across lots of things that I didn't even know were possible with Java. In this weekly blog series I will go through 11 of these 'crazy learnings' that surprised me the most, even as an experienced developer. This week we'll focus on anonymous subclasses in enums.

## Very tidy source files

Most enums I've created are very tidy source files. They contain a few enum constants, a String representation and perhaps an enum constructor. Most of the times, that's *it*.

When I was doing a few practice exams to prepare for my Java certification, I came across a few enum definitions that were a lot more extensive than the ones I was used to: *they contained anonymous subclasses*. In this blog post I'll give an example of this phenomenon and elaborate on the use (and misuse) of it.

## Conference enum

Let's extend the code domain we used in previous installments of the 'Eleven crazy learnings' blog series, and define a `Conference` enum:

```java
public enum Conference {
    JAVA_ONE("JavaOne", 2022, "the USA"),
    J_FALL("J-Fall", 2022, "the Netherlands"),
    DEVOXX_UK("Devoxx UK", 2023, "the United Kingdom");

    private final String name;
    private final Year nextEdition;
    private final String country;

    Conference(String name, int nextEdition, String country) {
        this.name = name;
        this.nextEdition = Year.of(nextEdition);
        this.country = country;
    }
}
```

So for a single conference we store a name, year and country.
Next, we'll add a method that returns a `String`: it has the purpose of giving us some information on the next occurrence of a specific `Conference`:

```java
public String whenIsTheNext() {
    return String.format("The next %s will be in %d; it will take place in %s.",
            name, nextEdition.getValue(), country);
}
```

And to make sure that this code is working in the way we want it, we have written a few test cases (before we wrote the code, of course - TDD for the win) to assert the correct behaviour.

```java
@Test
@DisplayName("whenIsTheNext() for DEVOXX_UK should inform us of the year and place of the next Devoxx UK")
void whenIsTheNextDevoxxUk() {
    assertThat(Conference.DEVOXX_UK.whenIsTheNext()).isEqualTo(
            "The next Devoxx UK will be in 2023; it will take place in the United Kingdom.");
}

@Test
@DisplayName("whenIsTheNext() for JAVA_ONE should inform us of the year and place of the next JavaOne")
void whenIsTheNextJavaOne() {
    assertThat(Conference.JAVA_ONE.whenIsTheNext()).isEqualTo(
            "The next JavaOne will be in 2022; it will take place in the USA.");
}
```

Now, for the J-Fall case, I want to change the behaviour slightly.
As most of you may already know, J-Fall is the best one-day Java conference on the planet.
Don't just take my word for it, [a few](https://twitter.com/hansolo_/status/786514484017950720?s=20&t=xOYqkjOJXAOzd7cN2ai5Wg) [other Java folks](https://twitter.com/KoTurk77/status/1456174204525686789?s=20&t=qVlzQ4QBNymS4Ex1R6vpPQ) tend to agree with me.

So with good reasons, I added the following test case for J-Fall:

```java
@Test
@DisplayName("whenIsTheNext() for J_FALL should show our love for this fantastic conference!")
void whenIsTheNextJFall() {
    assertThat(Conference.J_FALL.whenIsTheNext()).isEqualTo(
            "The next J-Fall will be in 2022; it will take place in the Netherlands." +
                    " It is the best one-day conference we know!");
}
```

Well, obviously the test case will fail, because we currently use the same implementation of the `whenIsTheNext()` method for all values of the enum. So, let's change that. 

Now, before I got my Java 11 certification, I would probably have added an ugly if statement in the body of the `whenIsTheNext()` method, like this:

```java
// Gets the job done, but looks ugly and is not maintainable.
public String whenIsTheNext() {
    String whenIsTheNext = String.format("The next %s will be in %d; it will take place in %s.", name, nextEdition.getValue(), country);
    
    if (this == J_FALL) {
        whenIsTheNext += " It is the best one-day conference we know!";
    }
    
    return whenIsTheNext;
}
```

Although this will get the job done, it looks ugly and it's not very maintainable.
While studying for the Java 11 certification I learned that you can override enum methods for a specific enum constant if you provide an anonymous subclass at the initialisation site:

```java
// This is a lot better.
JAVA_ONE("JavaOne", 2022, "the USA"),
J_FALL("J-Fall", 2022, "the Netherlands") {
    @Override
    public String whenIsTheNext() {
        return super.whenIsTheNext() + " It is the best one-day conference we know!";
    }
},
DEVOXX_UK("Devoxx UK", 2023, "the United Kingdom");
```

So although I never heard about this feature until recently, it can still help you out in some specific cases.
Though I wouldn't go overboard with it by introducing anonymous subclasses in enum definitions for all possible variants of your enum logic.
Use it appropriately, just as you would use inheritance in some specific cases (but certainly not everywhere!).

![Conference](/images/blog/conference.jpg)
> Image from <a href="https://pxhere.com/nl/photo/489447">PxHere</a>

## Other blog posts in this series

Did you miss a blog post in this series? Here's a list of all posts that have been published so far:

1. [A few freaky array declarations](/2022/06/28/eleven-crazy-learnings-initialising-arrays.html)
2. [Stream elements should implement Comparable](/2022/07/05/eleven-crazy-learnings-stream-elements-comparable.html)
3. [Accessing static interface methods](/2022/07/12/eleven-crazy-learnings-accessing-static-interface-methods.html)
4. **Anonymous subclasses in enums**
5. [Division by zero]({% post_url 2022-07-26-eleven-crazy-learnings-division-by-zero %})
6. [Method overloading priorities]({% post_url 2022-08-02-eleven-crazy-learnings-method-overloading-priorities %})
7. [The crazy stuff that is allowed in switch statements]({% post_url 2022-08-09-eleven-crazy-learnings-crazy-stuff-in-switch-statements %})
8. [Equality in cloned arrays]({% post_url 2022-08-16-eleven-crazy-learnings-equality-in-cloned-arrays %})
9. [Wrapper objects: some are more equal than others]({% post_url 2022-08-23-eleven-crazy-learnings-wrapper-objects-some-are-more-equal-than-others %})
10. [Functional interfaces actually CAN contain multiple abstract methods]({% post_url 2022-08-30-eleven-crazy-learnings-functional-interfaces %})