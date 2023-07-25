{% assign unconferenceTalks = include.talks | where: "type", "unconference" %}
{% include talks/talkType.md anchor="unconference" title="Unconference Talks" talks=unconferenceTalks %}