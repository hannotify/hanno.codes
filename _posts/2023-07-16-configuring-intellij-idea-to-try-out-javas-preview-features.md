---
layout: post
title: "Configuring IntelliJ IDEA To Try Out Java's Preview Features"
date: 16-07-2023 21:30:00 +0200
header:
  teaser: /assets/images/blog/configuring-intellij-idea-for-trying-out-javas-preview-features/settings.jpg
excerpt: This is how I configured my IntelliJ IDEA to be able to try out Java's preview features.
tags: 
- java
- intellij-idea
---

Currently I'm working on both an [article](/articles/) and a [talk](/talks/#javas-concurrency-journey-continues-exploring-structured-concurrency-and-scoped-values) on structured concurrency in Java.
Most of the work I did until now was based on [JEP 437](https://openjdk.org/jeps/437), which has been in Second Incubator status in Java 20.
Structured concurrency is now set to appear in Preview Status as part of Java 21 (see [JEP 453](https://openjdk.org/jeps/453)), and I was eager to try out the feature and see what had changed since Java 20.
It surprised me how much configuration I needed to do to get IntelliJ IDEA to play nice with this preview feature.
Here's an overview of what I did to get it working.

![Settings](/assets/images/blog/configuring-intellij-idea-for-trying-out-javas-preview-features/settings.jpg)
> Image by Pixabay from <a href="https://www.pexels.com/photo/settings-android-tab-270700/">Pexels</a>

## Downloading an Early-Access Build of JDK 21

I started out by downloading an early-access build of JDK 21.
Using [SDKMAN!](https://sdkman.io/), this only takes like a minute or so.
Running `sdk list java` lists all JDK binaries that are currently available.
This time, I opted for the latest early-access build of OpenJDK 21, so I ran:

```bash
sdk install java 21.ea.31-open
```

...and all was well.

## Updating the IDE

Then, I updated IntelliJ IDEA to the latest version (which is `2023.1.4` at the time of writing).
This is a rather crucial step, because older versions usually don't have support for the latest Java features.

## Setting the SDK and Language Level

Then, I opened the Module Settings (`⌘+↓` or `Ctrl+Alt+Shift+S` if you prefer shortcuts) and changed the SDK to OpenJDK 21 (the one I just downloaded). 
And I set the 'Language level' to 'X - Experimental features'.
Note that the 'Modules' tab has a separate setting for 'Language level', so I set that to 'X - Experimental features' as well.

![Configuring Module Settings](/assets/images/blog/configuring-intellij-idea-for-trying-out-javas-preview-features/x-experimental-features.png)

## First Try

After that I wrote some [concurrent code](https://github.com/hannotify/structured-concurrency-bar) that used a `StructuredTaskScope`, created a `Main` class and ran its `main()` method just to give it a try.
Then this happened:

```
java: java.util.concurrent.StructuredTaskScope is a preview API and is disabled by default.
  (use --enable-preview to enable preview APIs)
```

Which made sense of course, as I hadn't allowed the use of preview features yet in any of the configuration.

## Enabling Preview Features in the Run Configuration

So I opened the run configuration that belonged to the `Main` class to add the `--enable-preview` flag to the VM options.
In recent versions of IntelliJ IDEA the 'Add VM options' functionality has been hidden underneath the 'Modify options' dropdown.

![Add VM options](/assets/images/blog/configuring-intellij-idea-for-trying-out-javas-preview-features/add-vm-options.png)

Upon activating 'Add VM options' a 'VM options' text field appeared, in which I entered `--enable-preview`.

## Second Try

After applying the run config changes, I ran the `Main` class again.

```
java: java.util.concurrent.StructuredTaskScope is a preview API and is disabled by default.
  (use --enable-preview to enable preview APIs)
```

Umm, say what?
That didn't make sense to me. 
I took some time to research the issue, and according to this [StackOverflow post](https://stackoverflow.com/questions/72083752/enable-preview-features-in-an-early-access-version-of-java-in-intellij), I had forgotten to enable the preview features on a per-module basis.

## Enabling Preview Features Per-Module

In order to do so, I popped open 'Settings' (`⌘+,` or `Ctrl+Alt+S` if you prefer shortcuts), went to 'Build, Execution, Deployment' > 'Compiler' > 'Java Compiler' and added `--enable-preview` in the little table that's directly beneath 'Override compiler parameters per-module'.

> You can edit the cells in this table by double-clicking.

![Override compiler parameters per-module](/assets/images/blog/configuring-intellij-idea-for-trying-out-javas-preview-features/per-module-settings.png)

## Final Try

And that's when things started working!
When I ran the `Main` class again, I finally got the output I was expecting.
Why this configuration is (seemingly) so complicated is beyond me.
If you happen to know more about it, feel free to let me know!
And until that happens: have fun playing around with Java's preview features in IntelliJ IDEA!
