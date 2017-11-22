---
layout: default
---

![](images/hanno-at-devoxx-cropped.jpg)

# [](#bio)Bio

Hanno is a Java Developer, Scrum Master and Trainer at Info Support (Veenendaal, Netherlands). He has over 10 years of experience developing enterprise software in various fields (insurance companies, banks, hospitals, industry) and currently works for the Dutch Railway Company (‘NS’). He loves building innovative software and has a passion for clean, elegant solutions. On top of that, he likes continuous delivery, behavior-driven development and all things agile.

# [](#upcoming-events)Upcoming events

{% capture nowUnix %}{{'now' | date: '%s'}}{% endcapture %}
{% capture eventTime %}{{event.dateStart | date: '%s'}}{% endcapture %}

{% for event in site.data.events %}
    {% if eventTime > nowunix %}
        ## [](#{{event.id}})[{{event.name}}]({{event.url}})

        |-------------:|:-------------------|
        | **Location** | {{event.country}}, {{event.city}} |
        |     **Date** | {{event.dateStart}} - {{event.dateEnd}}  |
        |     **Talk** |  |        
    {% endif %}
{% endfor %}

## [](#jvmcon-2018)[JVMCon 2018](https://jvmcon.com)

|-------------:|:-------------------|
| **Location** | Netherlands, Ede   |
|     **Date** | 30th January 2018  |
|     **Talk** | [Building a Spring Boot Application: Ask the Audience!](talks#building-a-spring-boot-application-ask-the-audience) |

# [](#past-events)Past events

## [](#devoxx-2017)[Devoxx 2017](https://devoxx.be)

| **Location** | Belgium, Antwerpen |
| **Date**     | 6th - 10th November 2017  |
| **Talks**    | [QWERTY or DVORAK? Debunking the Keyboard Layout Myths](talks#qwerty-or-dvorak-debunking-the-keyboard-layout-myths) |
|              | [Slide Deck Version Management: The Good, The Bad and the Ugly](talks#slide-deck-version-management-the-good-the-bad-and-the-ugly) |

## [](#jfall-2017)[J-Fall 2017](https://jfall.nl)

| **Location** | Netherlands, Ede   |
| **Date**     | 2nd November 2017  |
| **Talks**    | [Building a Spring Boot Application: Ask the Audience!](talks#building-a-spring-boot-application-ask-the-audience) |
|              | [Slide Deck Version Management: The Good, The Bad and the Ugly](talks#slide-deck-version-management-the-good-the-bad-and-the-ugly) |

## [](#jbcnconf-2017)[JBCNConf 2017](https://jbcnconf.com)

| **Location** | Spain, Barcelona      |
| **Date**     | 19th - 21th June 2017 |
| **Talk**     | [Building a Spring Boot Application: Ask the Audience!](talks#building-a-spring-boot-application-ask-the-audience) |

## [](#javaland-2017)[JavaLand 2017](https://javaland.eu)

| **Location** | Germany, Brühl          |
| **Date**     | 28th - 30th March 2017  |
| **Talk**     | [Building a Spring Boot Application: Ask the Audience!](talks#building-a-spring-boot-application-ask-the-audience) |

## [](#devoxx-uk-2016)[Devoxx UK 2016](https://devoxx.co.uk)

| **Location** | United Kingdom, London |
| **Date**     | 8th - 10th June 2016   |
| **Talk**     | [Migrating 25K lines of Ant scripting to Gradle](talks#migrating-25k-lines-of-ant-scripting-to-gradle)

## [](#jprime-2016)[jPrime 2016](http://jprime.io/nav/2016)

| **Location** | Bulgaria, Sofia      |
| **Date**     | 26th - 27th May 2016 |
| **Talk**     | [Migrating 25K lines of Ant scripting to Gradle](talks#migrating-25k-lines-of-ant-scripting-to-gradle)