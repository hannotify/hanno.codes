---
layout: post
author: Hanno Embregts & Peter Wessels
title: Pattern matching will make Java an even more powerful language
date: 10-06-2021 11:59:02 +0200
tags: 
- java
- pattern-matching
---

We've known lambdas and streams since Java 8, and they've made Java a more powerful language. In the next few versions of Java, even more features that originated in functional languages will be added, one of which is *pattern matching*. It provides an elegant way to apply conditions to certain aspects of an object. We set out to investigate the possibilities that were introduced in JEP 305 ('Pattern Matching for instanceof') and how the pattern matching roadmap will make Java an even more powerful language.

## Type patterns

We've probably all been there: you want to know whether an object is of a certain type, so that you can call one of its methods. To achieve this in 'classic' Java, you need an *instanceof* test, a cast to the target type and an assignment to a local variable. This approach comes with repetition, is sensitive to errors and - most importantly - not very fun to write. If only we could improve that!

[Pattern Matching for instanceof](https://openjdk.java.net/jeps/305), which is a preview feature in Java 14, is a first step in adding support for pattern matching to Java. This feature allows us to replace the three steps we mentioned earlier by just a single expression!

```java
// Java 13
@Override
public boolean equals(Object o) {
    // As pattern matching is music to our ears, we defined a domain model of an orchestra. 
    if (!(o instanceof Musical)) {
        return false;
    }
     
    Musical m = (Musical) o;
    return name.equals(m.name) && isPrincipal == m.isPrincipal;
}
 
// Java 14
@Override
public boolean equals(Object o) {
    return o instanceof Musical m && name.equals(m.name) && isPrincipal == m.isPrincipal;
}
```

This code example defines a pattern `Musical m`, that consists of a type `Musical` and a label `m`. This is the same type that is used in an *instanceof* test to match an object of said type. After a successful match the object is automatically cast and assigned to the pattern variable with a name equal to the defined label. This single pattern now replaces the *instanceof* test, an explicit cast and a variable assigment. 

In the near future [switch expressions will also start supporting pattern matching](http://openjdk.java.net/jeps/406), which really showcases its power. The exact syntax is still subject to change, but it will probably look like the example below.

```java
String whatDoesTheMusicianSay = switch (musical) {
    case Vocal v  -> String.format("The singer goes %s", v.sing()); 
    // The singer goes "Aaaa"
    case Guitar g -> String.format("The guitar goes %s", g.play());
    // The guitar goes "Pling pling"
    case Drums d  -> String.format("The drums go %s", d.hit());
    // The drums go "Padum Chee"
    default       -> String.format("Unknown Musical goes %s", obj.play()); 
};
```

As you can see, the incorporation of pattern matching into switch expressions can yield code that is surprisingly elegant. But wait, there's more.

## Constant patterns

A second kind of pattern may be already familiar to you: the case label of a switch statement. Case labels currently can take numeric, String or enum values, and in the future they will be referred to as *constant patterns*. It's a new name for a familiar concept, to further clarify that case labels will be able to take multiple kinds of patterns in the future. 

## Deconstruction patterns

*Deconstruction patterns* take pattern matching to the next level by adding an 'extract' capability after a successful pattern match. In the future we'll be able to use this feature, thereby eliminating the need to call any getters on the matched object. Instead, we can gather all revelant fields in a single statement. A deconstruction pattern performs this gathering by relying on a *pattern definition*, a 'reverse constructor' of some sort, to assign the values of the object's fields to the pattern variables. 

```java
return switch (musical) {
    case Orchestra(List<Musical> musicians) -> String.format("Orchestra, consisting of %d musicians.", musicians.size());
    // Using definition "public pattern Orchestra(List list)"
    case InstrumentFamily(Guitar(true, String guitarName), Trumpet(true, String trumpetName)) -> String.format("Two principals of guitarist %s and trumpetist %s.", guitarName, trumpetName);
    // Using definitions "public pattern InstrumentFamily(Musical m1, Musical m2)" and "public pattern Musical(boolean isPrincipal, String name)"
};
```

The patterns in this code example match on the types `Orchestra` and `InstrumentFamily`. The match on the `Orchestra` type will only succeed if the object `musical` can be extracted as a `List` of `Musical`s. If the match succeeds, the variable `musicians` will be in scope and we can interact with it. So a deconstruction pattern allows us to match objects, cast them and interact with its fields: all using a single case construct.

It can't get more powerful than this, you might think. But it can! The patterns that we wrote about until now can also be used in *composition*, as is depicted in the second case block of the code example. Here we combine one deconstruction pattern (`InstrumentFamily(...)`), a few type patterns (`Guitar(...)` and  `Trumpet(...)`) and two constant patterns (`true`). To summarize: this pattern matches any `InstrumentFamily` that can be extracted as a `Guitar` and a `Trumpet`, which both have `isPrincipal` set to `true`. This shows that when composing patterns all pattern kinds are at your disposal and can be combined as you wish.

## Var & any patterns

Java 10 brought us the capability of [declaring variables with the `var` keyword](https://openjdk.java.net/jeps/286), instead of a specific type. The same capability comes to mind with *var patterns*: pattern can use `var` instead of a specific type, and the compiler will then infer the right type pattern. This also bypasses the type condition, as the compiler will match on the first field that yields a valid type pattern, after which it will bind the encountered value to the pattern variable as usual.

```java
case Guitar(var name) -> String.format("The guitar is called %s", name);
```

*Any patterns* (denoted by a single underscore character) are like var patterns: they match the first field that is found, but after matching successfully no value will be bound to any variable. This may not sound very useful at first, but it can become a powerful tool when used in a pattern composition. In this role, an any pattern can express that some parts of a matched object are in fact irrelevant and can be ignored.

```java
case Orchestra(_, VocalFamily vf), Orchestra(VocalFamily vf, _) -> "This orchestra contains a vocalist.";
```

## A better serialization?

> We zagen eerder al dat een deconstruction pattern een objectstructuur om kan zetten in losse, getypeerde velden. Eigenlijk net zoals een constructor losse, getypeerde velden omzet in een objectstructuur. Ze zijn elkaars tegenovergestelde. En dat inzicht zou wel eens een betere versie van serialisatie kunnen opleveren. Serialisatie is belangrijke functionaliteit in Java, maar veel mensen hebben een hekel aan de huidige implementatie. Het ondermijnt bijvoorbeeld het toegangsmodel, de serialisatielogica is geen ‘leesbare code’ en het omzeilt constructors en bijbehorende datavalidatie. Maar het gebruik van patterns zou de situatie drastisch kunnen verbeteren.

> Wanneer klasses in de toekomst pattern-definities kunnen bevatten, zou dat een goede plek zijn om een instantie van die klasse te serialiseren. Deserialiseren gebeurt dan via een (overloaded) constructor of een factory-methode. Hiermee zou alle serialisatielogica ‘leesbare code’ worden en expliciet onderdeel van de klassedefinitie. Door de annotaties is het direct duidelijk waar de code voor dient. Ook zou bestaande datavalidatie eenvoudig aangeroepen kunnen worden.

> Het ondersteunen van serialisatie op meerdere versies van een klasse zal een uitdaging blijven, al bestaan er wel plannen om de @Serializer– en @Deserializer-annotaties met een version-property uit te rusten die een klasseversie kan bevatten [6]

```java
class Orchestra {
    @Deserializer
    public static Orchestra deserialize(Musical[] musicals) {
        return Orchestra.ensemble(musicals);
    }
 
    @Serializer
    public pattern Orchestra(Musical[] musicals) {
        musicals = this.musicalPeople.toArray();
    }
}
```

## Records & sealed types

> Dat pattern matching niet op zichzelf staat, maar deel uitmaakt van een groter plan wordt nog duidelijker als we naar records gaan kijken. Deze compacte manier om immutable data te modelleren werd eerder al geïntroduceerd in Java Magazine [7] en heeft als voordeel dat constructors, getters, equals()– en hashCode()-implementaties al beschikbaar zijn, zonder ze expliciet te definiëren. Wanneer deconstruction patterns beschikbaar komen in Java, zullen records ook automatisch pattern-definities bevatten [8], waarmee een record direct bruikbaar wordt als deconstruction pattern in een switch expression.

```java
record Trumpet(boolean isPrincipal, String name) implements Musical {
    // This code is generated:
    public pattern Trumpet(boolean isPrincipal, String name) {
        isPrincipal = this.isPrincipal;
        name = this.name;
    }
}
```

> Ook een andere, relatief nieuwe feature zal in de toekomst goed samenwerken met pattern matching, namelijk sealed types. Met sealed types – beschikbaar in preview vanaf Java 15 – leg je voor een interface of superklasse uitputtend vast welke implementaties er mogen zijn.

```java
sealed interface Musical permits Vocal, Guitar, Drums {}
```

> Wanneer we vervolgens een switch expression op een sealed type toepassen, zal de default branch van de switch niet meer nodig zijn. De compiler ‘weet’ immers dat Musical maar drie implementaties heeft (en blijft hebben).

```java
String whatDoesTheMusicianSay = switch (musical) {
    case Vocal v -> String.format("The singer goes %s", v.sing()); 
    // The singer goes "Aaaa"
    case Guitar g -> String.format("The guitar goes %s", g.play()); 
    // The guitar goes "Pling pling"
    case Drums d -> String.format("The drums go %s", d.hit()); 
    // The drums go "Padum Chee"
};
```

## Wrap-up

We have seen that pattern matching can do a lot more than just replace a few casts at an *instanceof* test. It improves switch expressions, it's able to express complex logic elegantly and deconstructing objects can be a breeze, because pattern definitions are the opposite of constructors. On top of that, the design of new features like sealed types and records has incorporated support for pattern matching up front. This shows that pattern matching is quickly becoming a very important Java feature, and it is here to stay.

## References & acknowledgements

The full code examples can be found at [GitHub](https://github.com/MrFix93/pattern-matching-orchestra).

This blog post is based on an article that was published in the Dutch Java Magazine ([#2-2021](https://nljug.org/java-magazine/java-magazine-2-2021-brand-new/)) and has been published here with permission by the [Dutch Java User Group](https://nljug.org/).