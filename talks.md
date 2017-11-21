---
layout: default
---

# [](#regular)Regular Talks

{% assign regularTalks = site.data.talks | where: "type", "regular" %}
{% for regularTalk in regularTalks %}

## [](#{{ regularTalk.id }}){{ regularTalk.title}} 

{{ regularTalk.description }}

{% endfor %}

# [](#ignites)Ignites

{% assign igniteTalks = site.data.talks | where: "type", "ignite" %}
{% for igniteTalk in igniteTalks %}

## [](#{{ igniteTalk.id }}){{ igniteTalk.title}} 

{{ igniteTalk.description }}

{% endfor %}
