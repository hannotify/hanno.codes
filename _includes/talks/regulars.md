# [](#regular)Regular Talks

{% assign regularTalks = site.data.talks | where: "type", "regular" %}
{% for regularTalk in regularTalks %}

## [](#{{ regularTalk.id }}){{ regularTalk.title}} 

{{ regularTalk.description }}

{% endfor %}