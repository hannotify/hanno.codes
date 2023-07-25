{% assign activeTalks = site.data.talks | where: "retired", "false" %}

{% include talks/keynote.md talks=activeTalks %}
{% include talks/regular.md talks=activeTalks %}
{% include talks/lightning.md talks=activeTalks %}
{% include talks/unconference.md talks=activeTalks %}
{% include talks/ignite.md talks=activeTalks %}
