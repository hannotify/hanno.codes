{% assign lightningTalks = site.data.talks | where: "type", "lightning" %}
{% include talks/talkType.md anchor="lightning" title="Lightning Talks" talks=lightningTalks %}
