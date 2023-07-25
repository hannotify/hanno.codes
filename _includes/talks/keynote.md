{% assign keynotes = include.talks | where: "type", "keynote" %}
{% include talks/talkType.md anchor="keynote" title="Keynotes" talks=keynotes %}
