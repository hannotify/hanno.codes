# [](#ignites)Ignites

{% assign igniteTalks = site.data.talks | where: "type", "ignite" %}
{% for igniteTalk in igniteTalks %}

## [](#{{ igniteTalk.id }}){{ igniteTalk.title}} 

{{ igniteTalk.description }}

{% endfor %}