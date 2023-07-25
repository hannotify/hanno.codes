{% assign retiredTalks = site.data.talks | where: "retired", "true" %}
{% include talks/talkType.md anchor="retired" title="Retired" talks=retiredTalks %}
