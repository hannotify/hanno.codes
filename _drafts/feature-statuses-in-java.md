---
layout: post
title: "The Ultimate Guide To Feature Statuses in Java"
date: dd-05-2023
header:
  teaser:
excerpt: Never be confused about 'preview', 'experimental' or 'incubating' again!
tags: 
- java
---

One of my favourite ways to discover upcoming features to the Java programming languange is to keep track of [the Java Enhancement Proposals](https://openjdk.org/jeps/0) (or JEPs) that were recently added. 
The feeling you get when you discover a little gem like [Unnamed Classes and Instance Main Methods](https://openjdk.org/jeps/445) is just fantastic! 
But when the JEP title contains a feature status like 'incubator' or 'experimental', I always get somewhat confused about which status is which and what it means for the way I can use the feature.
To prevent even more confusion in the future (both for myself and for you), here's a guide to these feature statuses and what they mean.

## Four Different Feature Statusess

In Java, there are four different feature statuses that are used to indicate the level of maturity of a new feature: _final_, _preview_, _incubator_ and _experimental_. 
These statuses help developers understand the level of support and stability of a feature and whether it is ready for production use. 

## Final

...

Important safeguards prevent developers from using nonfinal features accidentally. This is necessary because a nonfinal feature may well be different when it becomes final and permanent in a later Java feature release. Moreover, only final, permanent features are subject to Java’s stringent backward-compatibility rules.

Therefore, to avoid unintentional use, preview and experimental features are disabled by default, and the JDK documentation unequivocally warns developers about the nonfinal nature of these features and any of their associated APIs.

## Preview

> Ensure that features are "done right" before they become final and permanent parts of the language by providing a suitable window of time in which features can be improved based on user feedback.

* fully specified and implemented
* impermanent
* provoke developer feedback based on real world use
* may lead to it becoming permanent
* communicates whether a new feature is "coming to Java" in approximately its current form within the next 12 months.
  * previewing may take longer than 12 months under certain circumstances
* code which uses preview features from an older release of the Java SE Platform will not necessarily compile or run on a newer release.
* not all new language, VM, and API features *have* to be available initially as preview features.
* 3 kinds of preview features exist:
  * language features
  * VM features
  * APIs
* a preview feature is present in all Java compilers and JVM implementations but disabled by default.
* The Java SE Platform has global reach, so the cost of a mistake in the design of a Java language feature, JVM feature, or Java SE API is high.
* That's why it is desirable for the feature to enjoy a period of broad exposure after its specification and implementation are stable but before it achieves final and permanent status in the Java SE Platform.
* Previewing a feature in the JDK will encourage tool vendors to build good support for the feature before the bulk of Java developers use it in production.
* Over the six-month course of the JDK feature release, and perhaps the following release too, the feature's "real world" strengths and weaknesses will be evaluated to decide if it has a long-term role in the Java SE Platform. 
* Ultimately, the feature will either be granted final and permanent status (with or without refinements) or be removed.

### How Complete Is A Preview Feature?

By "complete", we do not mean "100% finished", since that would imply feedback is pointless. Instead, we mean the preview feature meets two criteria:

1. (Readiness) The preview feature has a high probability of being 100% finished within 12 months. This timeline reflects our experience that two rounds of previewing is the norm, i.e., preview in Java $N and $N+1 then final in $N+2. For APIs that have exceptionally large surface areas or engage deeply with the JVM, and for language features that integrate with other language features as a matter of necessity, we anticipate additional rounds of feedback and revision, as such features will underpin the Java ecosystem for decades to come.
2. (Stability) The preview feature could credibly achieve final and permanent status with no further changes. This implies an extremely high degree of confidence in the concepts which underpin the feature, but does not completely rule out making "surface level" changes in response to feedback. 

### Running a Preview Feature

Preview features are specific to a given Java SE feature release and require the use of special flags at compile time as well as at runtime. In a given Java SE platform release (for example, Java 14), javac --enable-preview --release 14 … will enable the Java compiler to generate class files that use preview features. The compiler will also issue a warning to inform developers that preview features are being used. And similarly, java --enable-preview … will allow those classes to run on a matching JVM (version 14, in this case), while jshell --enable-preview will enable the use of preview features on the corresponding jshell version.

### Key Properties

The key properties of a preview feature are:

1. _High quality._ A preview feature must display the same level of technical excellence and finesse as a final and permanent feature of the Java SE Platform. 
2. _Not experimental._ A preview feature must not be experimental, risky, incomplete, or unstable. An experimental feature must not be previewed in a JDK feature release, but rather be iterated and stabilized in its own project, which in turn produces binaries that are clearly distinguished from binaries of the JDK Project. For the purpose of comparison, if an experimental feature in HotSpot is considered 25% "done", then a preview feature in Java SE should be at least 95% "done".
3. _Universally available._ The Umbrella JSR for the Java SE $N Platform enumerates the preview features of the platform. The desire for feedback and the expectation of quality means that these features are not "optional"; they must all be supported in full by every implementation of Java SE $N. In particular, all preview features must be disabled by default in a Java SE implementation, and an implementation must offer a mechanism to enable all preview features. 

### (According to ChatGPT)

The "preview" feature status is used for new features that are being introduced in a Java release, but are not yet finalized or fully supported. 
Preview features are intended to gather feedback from the community and to give developers an early look at the feature before it becomes stable. A preview feature is not intended for production use, and it may change or be removed in a future release. To use a preview feature, developers must enable it explicitly, either through command-line flags or other means.

Preview language features have also been effective at helping the community to adopt new features. We particularly wanted IDEs to support preview language features so that as many developers as possible could easily try them out, and IDE vendors have delivered strongly.

## Incubator

> A means of putting non-final APIs and non-final tools in the hands of developers, while the APIs/tools progress towards either finalization or removal in a future release.

* An incubating feature is an API or a tool, of non-trivial size, that is under development for eventual inclusion in the Java SE Platform or the JDK. The API or tool is not yet sufficiently proven, so it is desirable to defer standardization or finalization for a small number of feature releases in order to gain additional experience and feedback.
* An incubator module is a module in a JDK Release Project that offers an incubating feature: the module either exports an incubating API, or contains an incubating tool, or both.
* An incubator module is identified by the jdk.incubator. prefix in its module name, whether the module exports an incubating API or contains an incubating tool.
* If an incubating API is standardized in the Java SE Platform, or otherwise promoted to some stable mode of existence in the JDK, then its packages will be renamed and then exported from a non-incubator module. 
* If an incubating API is not standardized or otherwise promoted after a small number of JDK feature releases, then it will no longer be able to incubate, and its packages and incubator module will be removed.
* Applications on the class path must use the `--add-modules` command-line option to request that an incubator module be resolved. They are not resolved by default.
* Incubation applies to modules, and a preview feature is something that is more closely tight in with the language and the libraries.
* Also known as 'incubator modules'

JEP 11 introduces the notion of incubation to enable the inclusion of JDK APIs and JDK tools that might one day, after improvements and stabilizations, be included and supported in the Java SE platform or in the JDK. For example, the HTTP/2 Client API has been incubating—as a JDK-specific API in JDK 9 and JDK 10 via JEP 110—to finally leave that incubating phase and be included as a standard Java SE API in Java 11 (JEP 321).

incubating APIs are non-standard and delivered in standalone "incubator modules"
We will, however, continue to use incubation for APIs whose level of completeness and stability is considerably lower than that of preview APIs.

### Using Incubator Modules

Finally, incubator modules are also shielded from accidental use because incubating can be done only in the jdk.incubator namespace. Therefore, an application on the classpath must use the --add-modules command-line option to explicitly request resolution for an incubating feature. Alternatively, a modular application must specify requires or requires transitive dependencies upon an incubating feature directly.

### (According to ChatGPT)

The "incubator" feature status is used for features that are still under development and are not yet fully supported. Incubator features have passed the initial development and testing stages and are ready for wider use, but they are still subject to change before being promoted to a fully supported feature. Incubator features are intended for evaluation and feedback, and they may have different levels of support in different Java releases. Developers can use an incubator feature by enabling it explicitly, just like preview features.

## Experimental

An experimental feature must not be previewed in a JDK feature release, but rather be iterated and stabilized in its own project, which in turn produces binaries that are clearly distinguished from binaries of the JDK Project. For the purpose of comparison, if an experimental feature in HotSpot is considered 25% "done", then a preview feature in Java SE should be at least 95% "done". To make a further comparison, the level of completeness and stability expected for a preview API is considerably higher than the level expected for an incubating API.

Preview VM features are also distinct from "experimental" HotSpot features, which are early versions of low-level features that need to be explicitly unlocked in HotSpot at run time. (For example, when HotSpot's Z Garbage Collector was experimental in JDK 11, it was enabled via java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC.)

An experimental feature is a test-bed mechanism used to gather feedback on nontrivial HotSpot enhancements. Unlike JEP 12 for preview features, there is no JEP governing experimental features; the process for experimental features is more an established HotSpot convention than a formal process.

Let’s take the Z Garbage Collector as an example. ZGC offers a low-latency garbage collection pause time—below 10 ms but typically more around 2 ms—regardless of the heap size, even if the heap is as small as a few megabytes, or as large as multiple terabytes.

The ZGC team leveraged the experimental feature mechanism several times, with ZGC initially introduced in JDK 11 as an experimental feature limited to Linux x64 (JEP 333). Since then, additional improvements were added to ZGC (for example, concurrent class unloading, memory uncommit, and additional platforms) while other ZGC capabilities were ironed out.

The overall feedback and experience collected during those iterations enabled ZGC to be gradually solidified to a point where it now has the high quality expected for a HotSpot feature. Consequently, JEP 377 is now proposing to formally turn ZGC into a regular HotSpot production feature in JDK 15.

...

### Using Experimental Features

Experimental features are JVM features and are disabled by default; the -XX:+UnlockExperimentalVMOptions flag instructs HotSpot to allow experimental features. The actual experimental feature can then be enabled via specific flags, for example, -XX:+UseZGC for ZGC.

### (according to ChatGPT)

The "experimental" feature status is used for features that are highly experimental and not yet fully developed. Experimental features are still in the early stages of development and have not been thoroughly tested. These features may be incomplete, unstable, or have limited functionality. Experimental features are not recommended for use in production environments and may change or be removed in future releases. To use an experimental feature, developers must enable it explicitly and accept the associated risks.

## What about 'Second', 'Third' Et Cetera?

...

we've learned that large language features might need to preview more than twice, especially if they interact with other language features which are themselves in preview. For example, pattern matching for switch has previewed twice on its own merits, then twice more because of interactions with another, newer, preview feature (record patterns). Previewing three or more times reflects the ambition that all language features work together smoothly, and is incomparably better than trying to "hold the train" until all connected features are equally ready to preview.

## The Differences Summarised In Table Form


|              | Final | Preview           | Incubator    | Experimental |
|--------------|-------|-------------------|--------------|--------------|
| Completeness | complete | (almost) complete | incomplete   | incomplete   |
| Typical eature type | n/a | Language feature | API feature | HotSpot feature |
| Purpose | n/a | provoke developer feedback based on real world use | .. | .. |
| Distribution | regular | regular           | separate module(s)? | separate module(s) |
| Permanent    | yes | no                | no           | no               |
| How to use   | n/a | --enable-preview  | --enable-preview --add-modules <module-name> | ... -XX ... |

## History of a Few Features in Table Form

| Feature                     | Java 17 | Java 18     | Java 19     | Java 20     | Java 21      | 
|-----------------------------|---------|-------------|-------------|-------------|--------------|
| Pattern Matching for Switch | Preview | 2nd Preview | 3rd Preview | 4th Preview | Final        |
| Record Patterns             |         |             | Preview     | 2nd Preview | Final        |
| Scoped Values               |         |             |             | Incubator   | Preview      |
| Generational Shenandoah     |         |             |             |             | Experimental |

## Conclusion

In conclusion, the preview, incubator, and experimental feature statuses are used in Java to indicate the level of maturity of a new feature. Preview features are not yet fully supported, incubator features have reached a higher level of maturity, and experimental features are highly experimental and not yet fully developed. Developers should be aware of these feature statuses and use them appropriately based on their level of maturity and intended use.

## References

* [JEP 12: Preview Features](https://openjdk.org/jeps/12)
* [JEP draft: Preview Features: A Look Back, and A Look Ahead](https://openjdk.org/jeps/8300604)
* [The role of preview features in Java 14, Java 15, Java 16 and beyond](https://blogs.oracle.com/javamagazine/post/the-role-of-preview-features-in-java-14-java-15-java-16-and-beyond)