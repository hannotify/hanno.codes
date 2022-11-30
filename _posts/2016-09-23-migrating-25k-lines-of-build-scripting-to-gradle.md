---
layout: post
title: Migrating 25K lines of build scripting to Gradle
date: 2016-09-23 11:13:29 +0200
tags: 
- gradle
- ci-cd
---

Software developers are not particularly keen on maintaining build scripts. Sure, they will do their part in making things work and they’ll even have a go at optimizing the most necessary parts. But they prefer to spend their time writing production code using a ‘real language’, because they feel it better supports their desire for maintainability, flexibility and elegance. That old project that’s been gathering dust for ages and relies on thousands lines of Ant code is a case in point.

However, in the past years numerous new build tools have emerged, apart from industry standard Maven, which all focus on bringing more ‘real language’ features to build scripts. One of these tools is [Gradle](https://www.gradle.org/), which focuses on providing both structure and flexibility. On top of that, it features incremental building and provides excellent multi-module support.

During a project I did at [NS](https://www.ns.nl/) (Dutch Railways), we decided to dust off our massive Ant code base and tried to switch to Gradle, with the purpose to introduce maintainability, flexibility and elegance to our build code. This is our migration story ‘from the trenches’.

## Previous migration

To be really honest, this actually wasn’t the first build tool migration we tried to perform. A few months earlier we considered Maven as the target build tool. Due to Maven’s structure and the concept of *lifecycles*, we expected that our build files would become much clearer to developers; especially the ones who were new to the project and had been struggling to get a grip on the overflow of Ant targets that existed.

However, during the first stages of the migration we discovered that Maven’s mandatory structure didn’t really fit the Ant setup of our project. The only way we would get things working was by circumventing Maven’s conventions and put in a few extra hours doing a lot of custom XML configuration. But this didn’t exactly produce elegant code; far from it, actually! So we had no choice but to abandon the migration and try out a different build tool.

## A massive challenge

Getting to know Gradle was the easy part, with the internet as an endless supplier of greenfield tutorials. However, the project I mentioned was anything but a greenfield project. It contained almost a million lines of code and there were 30 full-time developers working on it. On top of that, the system was designed as a monolith and during continuous integration it behaved as such. Given these facts, performing a successful migration while still supporting the remaining Ant code proved to be a massive challenge. It occurred to us we would need a great migration strategy to pull it off.

## Migration strategy

So that’s what we came up with. Before writing any Gradle code, we did a few meetings in which we discussed the characteristics of the project and in what ways the project could benefit from this migration. This is what we came up with:

* Divide the project into several smaller components with clearly defined responsibilities.
* Start the migration with a small, isolated component that doesn’t depend on any other component.
* Migrate on a separate branch in parallel to regular development, so that the latter won’t suffer.
* Verify after each migration step that:
  * results are exactly the same as before;
  * no problems occur in existing Ant code.
* Not every single line of Ant code should be replaced;
  * focus on the parts that developers use on a daily basis;
  * Ant projects are ‘first class citizens’.

## Starting out

So with a massive heap of Ant code in one hand, and our brand-new migration strategy in the other, we got started by defining our first component: a generic part that was responsible for writing train data to the database. For future reference, let’s call this “Component A”. We picked Component A to be the first, because it didn’t depend on any other parts of the system. In fact, most of other parts of the system actually depended on Component A. So as long as we made sure the resulting JAR file of Component A appeared in Nexus (after bumping the version number of course), nothing else in the project could break because of this change. And the rest of the project would be able to decide for themselves when to use the new version that had been built with Gradle.

After Component A had been migrated, we chose a part of the system that only had the one dependency: to Component A. Let’s call this one “Component B”. After that, we could pick any component that had a dependency to either Component A or Component B, or to both of them. And on we went, rinse and repeat, until all components were defined and built with Gradle code.

## Challenges

As we expected, we hit a few bumps in the road. We will have a look at three of the most challenging problems we faced, and how we tried to solve them.

### Challenge #1: Dependency spaghetti

Migrating a component started by assembling all dependency information in a single Gradle build file. Unfortunately, dependency definitions were completely scattered throughout the Ant code. Moreover, the Ant scripting used all kinds of ‘temporary library directories’, so we found JARs everywhere; and different versions of them, too!

The root cause for this madness was the fact that some parts of the projects were over 10 years old and used to rely on a home-grown dependency management system (this was before we used Ivy for dependency management). This self-made system obviously didn’t support transitive dependencies, hence all the duplicates. So we tackled this problem by defining each dependency exactly once in the Gradle file and introducing transitivity to prevent any duplicates or conflicting versions.

### Challenge #2: Collaboration with existing Ant code

Some of the Ant code was meant to be left behind, because it was used once a week or less (for a deployment to an acceptance or production environment, for example). But after the first components were migrated to Gradle, we discovered that the remaining Ant code produced errors when executed for artifacts that were produced by Gradle. After a few hours of analysis, we discovered that the Ant code relied on certain values in the MANIFEST.MF files, and Gradle had altered or even omitted some of these values. So we compared the resulting JARs and made sure the manifest headers were exactly the same (except for `Ant-Version` and `Gradle-Version`, obviously).

### Challenge #3: Continuous integration & delivery

We expected some configuration changes in Jenkins to run all code integration through Gradle instead of Ant, but it proved to be more complex than that. Gradle support in Jenkins is excellent, but we didn’t exactly love the Gradle plugins that we had started to use to calculate our code metrics (using Jacoco, FindBugs and Sonar). A direct copy of the equivalent Ant plugins did nothing at all; it seemed we had some custom configuration in the Ant plugins that didn’t translate well. Most Gradle plugins can be tweaked to comply with custom requirements, but the documentation that came with these particular plugins wasn’t nearly as good as the documentation of Gradle itself. So we pushed some buttons and did a little trial and error, and suddenly it worked.

## Migration result

After tackling these challenges, we reached our migration goals and discovered that our project had improved greatly on the following:

* A component’s responsibility has become clearer;
* A build will only run if the particular component has changed;
* Run unit tests in parallel (Gradle decides when);
* Dependencies behave transitively.

The following table summarizes the lines of code that we had in our project before and after the migration:

| Language | Lines of code (before) | Lines of code (after) |
|--------|--:|--:|
| Ant	 | over 25,000 | about 15,000 |
| Gradle | 0 | 1,232 |

## Should my project use Gradle? 

So now you’re probably wondering if you should also use Gradle in your project. If you’re starting a brand-new project, by all means do! Your build scripts will be smart *and* beautiful!

If your project has existed for a while, you should be cautious and ask yourself a few questions:

* Will you benefit from Gradles key features? (better performance, maintainability, less verbosity, …)
* If so, is there any technical debt to solve?
    * use an artifact repository and remove duplicates;
    * divide your project into multiple components;
    * define a clear structure in your build logic;
    * stick to default configuration wherever you can.

If you’ve answered these questions and did all of the above, then you’ll be more prepared for the migration than we ever were. Happy migrating!

## Further reading

* [“Why Build Your Java Projects with Gradle Rather than Ant or Maven?”](http://www.drdobbs.com/jvm/why-build-your-java-projects-with-gradle/240168608) by Benjamin Muschko.
* [Gradle User Guide](https://docs.gradle.org/current/userguide/userguide.html)

> Cross-posted with permission. The original post is at <https://blogs.infosupport.com/migrating-25k-lines-of-ant-scripting-to-gradle/>