{% assign keynotes = site.data.talks | where: "type", "keynote" %}
{% include talks/talkType.md anchor="keynote" title="Keynotes" talks=keynotes %}
