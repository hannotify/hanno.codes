---
layout: post
title: Community Contributions
date: 10-02-2023
header:
  teaser:
tags: 
- community-contributions
---

{% capture nowUnix %}{{'now' | date: '%s'}}{% endcapture %}
{% assign pastEvents = "" | split: "" %}

{% for event in site.data.events %}
    {% if event.dateEnd %}
        {% capture eventTime %}{{event.dateEnd | date: '%s'}}{% endcapture %}
    {% else %}
        {% capture eventTime %}{{event.dateStart | date: '%s'}}{% endcapture %}
    {% endif %}

    {% if event.dateStart and nowUnix > eventTime %}
        {% assign pastEvents = pastEvents | push: event %}
    {% endif %}
{% endfor %}

{% assign pastEventsByYear = pastEvents | group_by: "year" %}

{% for eventByYear in pastEventsByYear %}
    {% for event in eventByYear.items %}
        {% for appearance in event.appearances %}
            {% assign talk = site.data.talks | where: "id", appearance.id | first %}   

{% if event.city %}
    {% assign location = event.city | prepend: " in " | append: ", " | append: event.country.name %}
{% else %}
    {% assign location = "(online)" %}
{% endif %}

On {{event.dateStart}} I presented a talk called "{{talk.title}}" at {{event.name}} {{event.year}} {{location}}.

Conference schedule: <{{appearance.sessionPageUrl}}>
        {% endfor %}
    {% endfor %}
{% endfor %}
