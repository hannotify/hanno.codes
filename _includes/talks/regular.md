{% assign regularTalks = include.talks | where: "type", "regular" %}
{% include talks/talkType.md anchor="regular" title="Regular Talks" talks=regularTalks %}
