---
layout: post
author: Hanno Embregts
title: The ultimate use case for text blocks
#date: 29-10-2021 15:31:21 +0200
tags: 
- java
---

[Text blocks](https://openjdk.org/jeps/355) have been part of the Java language since version 13 (when the feature was in preview) and I have talked about them on several occasions.
I have done countless live demos indicating their usefulness for SQL queries, HTML snippets or JSON content, for example.
But it wasn't until last week that I discovered text blocks at the very peak of their usefulness. 
By using them for regular expressions.

# Guitar descriptions

When doing live demos I prefer to use [music as an exciting code example](https://github.com/hannotify/pattern-matching-music-store/blob/main/src/main/java/com/github/hannotify/patternmatching/musicstore/guitars/Guitar.java), so while I was demonstrating text blocks I used to resort to defining multi-line descriptions for guitars, for example.

```java
var description = """
        The Fender Stratocaster, colloquially known as the Strat, is a model of electric guitar designed from 1952 into 1954 by Leo Fender, Bill Carson, George Fullerton, and Freddie Tavares.
        The Fender Musical Instruments Corporation has continuously manufactured the Stratocaster since 1954.
        It is a double-cutaway guitar, with an extended top "horn" shape for balance.
        Along with the Gibson Les Paul, Gibson SG, and Fender Telecaster, it is one of the most-often emulated electric guitar shapes.
        "Stratocaster" and "Strat" are trademark terms belonging to Fender.
        """;
var strat = new Guitar("Fender Strat", GuitarType.STRATOCASTER, description);
```

The problem here is that a typical guitar description doesn't contain many punctuation marks.
As a consequence in this case the benefits of a text block are not immediately clear.

# SQL, HTML and JSON

A far better example of using text blocks to their full potential is by feeding them some SQL, HTML or JSON, for example.

```java
String classicJsonString = "[\n" +
        "\t{\n" +
        "\t\t\"BRAND\" \t\t: \"Fender\",\n" +
        "\t\t\"PRODUCTNAME\" \t: \"American Special Stratocaster\",\n" +
        "\t\t\"GRAPHICNAME\"\t: \"American Special Stratocaster\",\t\t\n" +
        "\t\t\"COSTPRICE\"\t\t: \"900.00\",\n" +
        "\t\t\"MRSP\"\t\t\t: \"1299.99\",\n" +
        "\t\t\"QTYONHAND\"\t\t: \"7\",\n" +
        "\t\t\"QTYONBACKORDER\": \"2\",\n" +
        "\t\t\"DESCRIPTION\"\t: \"Comfortable chording throughout the neck and effortless bending via the modern radiuses Maple fingerboard, 22 jumbo frets and 'C' shape neck profile. Enjoy tonal flexibility with sparkling high end as well as thick overdrive from Fender's Grease bucket tone circuit which allows the player to roll off high end timbres without creating additional emphasis on the bass tones. Express yourself with variations in note pitch via the Synchronized Tremolo that has been a hallmark of the Stratocaster. Dial in a variety of ferocious tones as well as dirty bell like chimes via the 5-way selector switch and 3 Texas Special single coils pickups. Deluxe Gig Bag Included.\"\n" +
        "\t}\n" +
        "]";
String textBlockJson = """
        [
            {
                "BRAND"         : "Fender",
                "PRODUCTNAME"   : "American Special Stratocaster",
                "GRAPHICNAME"   : "American Special Stratocaster",		
                "COSTPRICE"     : "900.00",
                "MRSP"          : "1299.99",
                "QTYONHAND"     : "7",
                "QTYONBACKORDER": "2",
                "DESCRIPTION"   : "Comfortable chording throughout the neck and effortless bending via the modern radiuses Maple fingerboard, 22 jumbo frets and 'C' shape neck profile. Enjoy tonal flexibility with sparkling high end as well as thick overdrive from Fender's Grease bucket tone circuit which allows the player to roll off high end timbres without creating additional emphasis on the bass tones. Express yourself with variations in note pitch via the Synchronized Tremolo that has been a hallmark of the Stratocaster. Dial in a variety of ferocious tones as well as dirty bell like chimes via the 5-way selector switch and 3 Texas Special single coils pickups. Deluxe Gig Bag Included."
            }
        ]
        """;
```

Here the text blocks feature makes the JSON snippet far more readable than when it was enclosed in several classic String literals that had to be concatenated. 
Though one could argue that this is still not a very realistic example, because JSON values in real life would probably come from a REST interface and there would be no need to define them literally like we've done in the code example above.
It could perhaps work when defining test data, but its benefit would be somewhat limited when it's only usable in the test scope.

# Learning Kotlin

So in my live demos I was never really able to drive the point home that text blocks are a game changer.
And I was quite frustrated by that.
As the frustration slowly ebbed away, I started focusing on other things.
One of them was taking an "Advanced Kotlin" course at [Info Support](https://www.infosupport.com/en/home).
Because it was an 'advanced' course, I brushed up on my Kotlin skill by doing a few of the excellent [Kotlin Koans](https://kotlinlang.org/docs/koans.html) as created by JetBrains.
And [one of them](https://play.kotlinlang.org/koans/Introduction/String%20templates/Task.kt) was about using Kotlin's *string templates* with regular expressions.
At that moment something clicked inside my head and everything fell into place.

# The ultimate use case: regular expressions

Because of course the ultimate use case for Java text blocks are regular expressions!
Unlike SQL or JSON snippets, regular expression do regularly come up in production code.
And everyone knows they are pretty unreadable by themselves, let alone in a classic Java String where each backslash needs to be escaped.




(...)