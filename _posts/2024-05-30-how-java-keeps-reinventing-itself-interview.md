---
layout: post
title: How Java Keeps Reinventing Itself
date: 30-05-2024 20:30:00 +0200
header:
  teaser: /assets/images/blog/new-invention.jpg
excerpt: IT professionals sometimes get the impression that Java is a technically less advanced or outdated languange. But it turns out the exact opposite is true. I was interviewed on this topic by Info Support's AI interviewer 'Anne'. Keep reading to find out why Java is still relevant in this day and age.

tags: 
- java
- sustainability
---

I was interviewed by 'Anne', a custom version of GPT-4 that was developed at Info Support. Anne was provided with my [bio](/about-me) and the interview topic "Some people think Java is less relevant nowadays, but actually a lot of work is  being done to make sure the language remains modern and relevant." From that, Anne generated the first question and came up with seven additional ones based on the answers I gave. I think it turned out to be a good read, so I'm happy to share it here.

![New invention](/assets/images/blog/new-invention.jpg)
> Image from <a href="https://pxhere.com/en/photo/909565">PxHere</a>

## 🤖 Anne: What makes Java a relevant programming language in this day and age?

**Hanno:** I can think of several reasons. By far the most important is the release cadence that Java has been using since 2017. Every six months, a new version of Java is released. This means that new features become available to developers much faster than before.

Another reason has to do with the popularity of the JVM. It forms an abstraction layer on top of the operating system, and was originally designed to execute Java code on any platform. However, today there are several programming languages that work on the JVM, including Groovy, Scala, and Kotlin. This exposes the users of these languages to the JVM, which leads to a growing popularity of the Java platform as a whole.

## 🤖 Anne: How does the faster release cadence affect Java developers? Does it bring any challenges as well?

**Hanno:** Before 2017, a new Java version would not get released until all features that targeted that release were completed. Today new features become available every six months. I think this has a positive impact on developers' motivation, because they get to try out new things twice a year.

There are challenges, though. Larger features are now split into parts, and the corresponding APIs are often not yet finalized. These "preview features" require a special Java setting to be used. The goal is to quickly gather feedback and make necessary adjustments to the features. This also means that a preview feature in a later version may work differently in a subtle way. This is why the term "preview feature" was created, to remind developers that the feature might still change a bit.

## 🤖 Anne: Can you provide an example where the 'preview feature' approach has led to improvements based on feedback?

**Hanno:** In the case of the [Pattern Matching for switch](https://openjdk.org/jeps/441) feature, the designers initially proposed 'guarded patterns', which include an additional boolean expression after a pattern match. The syntax was a double ampersand (`&&`) in Java 18. Users provided feedback that they found this confusing, especially when the boolean expression itself also contained `&&` or `||` operations. So in Java 19, guarded patterns were replaced by 'when clauses', with the double ampersand being replaced by the keyword `when`.

## 🤖 Anne: Can you describe a situation where the JVM has contributed to solving a technical problem? And how do Scala and Kotlin contribute to the popularity of Java?

**Hanno:** The primary reason for the existence of the JVM has always been the ability to write software that works on all operating systems. The motto was 'write once, run anywhere'. Scala and Kotlin do not necessarily make Java as a language more popular. However, they do contribute to the popularity of the broader Java platform. Developers of these languages encounter the same tools and libraries, such as Maven, Spring, and Jenkins. This makes a potential transition to Java in the future easier. Additionally, the languages have a positive effect on each other. New Java features are often based on similar ideas from Scala and Kotlin, and vice versa.

## 🤖 Anne: How do you see the future of Java with the rise of Python and JavaScript? Will it remain relevant?

**Hanno:** JavaScript is mainly used for front-end development, so its use case differs from Java. Python, like Java, is intended for back-end development and has a large market share in data analysis. Java, on the other hand, is popular for web applications using REST and HTML, for example. So, the languages have each carved out their own niche. I believe they can coexist well and are unlikely to lose their relevance quickly.

## 🤖 Anne: What advice would you give to a beginner developer who wants to learn Java?

**Hanno:** This is a perfect time to start learning Java! Many useful constructs are currently being added to Java through [Project Amber](https://openjdk.org/projects/amber/), which make Java more powerful and versatile. The Java 21 feature [Unnamed Classes and Instance Main Methods](https://openjdk.org/jeps/445) is also relevant for beginners, as it makes it easier to write a first 'Hello World!' program. I expect these developments to significantly lower the barrier for beginner Java developers.

## 🤖 Anne: Which resources or learning paths do you recommend to master Java?

**Hanno:** The Oracle website [dev.java](https://dev.java) is a very good starting point. Here you can find tutorials, documentation, an online Java parser, and recordings by experts. The book [Effective Java by Joshua Bloch](https://a.co/d/83xQMBc) is also very valuable. I learned a lot from it. After you've grasped the basics of object-oriented programming with Java, I highly recommend to read this book.

## 🤖 Anne: How do you see the future of the IT industry and the role of Java in it?

**Hanno:** I am confident that Java will continue to keep pace with technological advancements. A good example of this is the strong support for generative AI that IDEs like IntelliJ IDEA and Visual Studio Code have added. Additionally, [Java is a relatively energy-efficient language](https://devm.io/java/energy-efficient-programming-languages-137264). Energy efficiency is becoming increasingly relevant in the climate crisis that is looming, and I believe the availability of sustainable energy will be crucial. Because the most sustainable energy is the energy you don't consume, I am convinced Java's energy efficiency will make the language even more relevant in the future.

## Acknowledgements

This interview was originally published in Dutch on the [Info Support website](https://carriere.infosupport.com/resources/hoe-java-zichzelf-opnieuw-uitvindt) and has been translated and published here with permission.
