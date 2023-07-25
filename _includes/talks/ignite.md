{% assign igniteTalks = include.talks | where: "type", "ignite" %}
{% include talks/talkType.md anchor="ignite" title="Ignites" talks=igniteTalks %}
