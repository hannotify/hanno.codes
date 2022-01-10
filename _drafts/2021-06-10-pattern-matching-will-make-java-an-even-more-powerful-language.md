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

> In het voorbeeld definiëren we een pattern Musical m, dat bestaat uit een type Musical en een label m. In een instanceof-test wordt het type gebruikt om een object van dat type te matchen. Nadat de match heeft plaatsgevonden, wordt het object automatisch gecast en toegekend aan de variabele met het gedefinieerde label. Zo vervang je met een enkel pattern de expliciete cast en het toekennen aan een variabele. In een equals-methode is de impact wellicht nog beperkt, maar de kracht wordt pas echt duidelijk wanneer we in de toekomst pattern matching kunnen gaan gebruiken in een switch-expressie [3]. De syntax kan nog wijzigen [4], maar een voorbeeld zie je hieronder.

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

> Je ziet het; het combineren van de nieuwe switch-expressie met pattern matching levert verrassend korte en bondige code op. En dit is nog maar het begin.

## Constant patterns

> Naast type patterns is een tweede soort pattern jullie al bekend: de constante case-labels in een switch-statement. De numerieke, String- en enum-waardes die daarbij horen noemen we in het vervolg constant patterns. Een nieuwe naam voor iets dat al lang bestaat, om zo duidelijker te maken dat de case-labels in de toekomst allerlei soorten patterns aankunnen.

## Deconstruction patterns

> Deconstruction patterns brengt de ondersteuning voor pattern matching nog een stapje verder door na het matchen op type, ook het object ‘uit te pakken’. Zo hoeven we niet meer afzonderlijke getters te gebruiken om interne velden te benaderen, maar kunnen we dat in een enkel statement. Zo’n deconstruction pattern gebruikt een pattern-definitie, een soort omgekeerde van een constructor, om bij matchen van een object de waardes van interne velden toe te kennen aan variabelen. In onderstaand codevoorbeeld zien we zo’n deconstruction pattern [4].

```java
return switch (musical) {
    case Orchestra(List musicians) -> String.format("Orchestra, consisting of %d musicians.", musicians.size());
    // Using definition "public pattern Orchestra(List list)"
    case InstrumentFamily(Guitar(true, guitarName), Trumpet(true, trumpetName)) -> String.format("Two principals of guitarist %s and trumpetist %s.", guitarName, trumpetName);
    // Using definitions "public pattern InstrumentFamily(Musical m1, Musical m2)" and "public pattern Musical(boolean isPrincipal, String name)"
};
```

> Zoals je herkent, matchen we in het voorbeeld op het type InstrumentFamily en Orchestra. Verder zien we dat bij het case-statement van type Orchestra de match alleen plaatsvindt als het genoemde veld uitgepakt kan worden als een lijst van Musical-objecten. Bij een match is deze variabele direct in scope en kunnen we er methodes op aanroepen. Met deze deconstruction patterns kunnen we objecten dus in een enkel case-statement matchen, casten en velden direct benaderen.

> Krachtiger dan dit wordt het niet, zou je denken. Behalve dan dat we alle beschreven patterns ook kunnen combineren, zoals staat weergegeven in het tweede case-statement van het codevoorbeeld hierboven. In dit geval combineren we een deconstruction pattern (het uitpakken van objecten bij een match), type patterns (het matchen op type) en een constant pattern (het matchen op een constante waarde). Samengevat matchen we op een InstrumentFamily als deze uitgepakt kan worden in een Guitar en Trumpet, waar voor beiden geldt dat isPrincipal gelijk is aan true. Zo staat de gehele collectie aan patterns tot je beschikking!

## Var & any patterns

> Lokale variabelen kunnen sinds Java 10 ook met het keyword var worden aangeduid, in plaats van met een expliciet type [5]. Hetzelfde principe ligt ten grondslag aan var patterns: we kunnen in onze patterns var gebruiken in plaats van het expliciete type, en laten de compiler het juiste type pattern afleiden. Er is daarmee geen conditie op het type meer actief; de compiler gebruikt het eerste veld dat een valide type pattern oplevert en bindt de bijbehorende waarde aan de variabele.

```java
case Guitar(var name) -> String.format("The guitar is called %s", name);
```

>Een any pattern (aangegeven met een underscore) is als een var pattern: het matcht op het eerstgevonden veld, maar bindt daarbij geen waarde aan de variabele. Dat klinkt misschien niet erg nuttig, maar als onderdeel van een genest pattern kan het elegant uitdrukken dat een deel van een component niet-relevant is en genegeerd kan worden.

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

> Pattern matching gaat een heel stuk verder dan alleen het voorkomen van casts bij een instanceof. Het verbetert switch expressions, het kan complexe logica elegant uitdrukken en het deconstrueren van objecten is een ‘first-class citizen’ geworden, doordat pattern-definities het tegenovergestelde zijn van constructors. Bovendien is bij het ontwerpen van nieuwe features als sealed types en records de ondersteuning van pattern matching alvast voorzien. Dat laat zien dat pattern matching een belangrijke feature in Java gaat zijn én blijven.

## References & acknowledgements

The full code examples can be found at [GitHub](https://github.com/MrFix93/pattern-matching-orchestra).

This blog post is based on an article that was published in the Dutch Java Magazine ([#2-2021](https://nljug.org/java-magazine/java-magazine-2-2021-brand-new/)) and has been published here with permission by the [Dutch Java User Group](https://nljug.org/).