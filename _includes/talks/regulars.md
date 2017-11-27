# [](#regular)Regular Talks

{% assign regularTalks = site.data.talks | where: "type", "regular" %}
{% for regularTalk in regularTalks %}

## [](#{{ regularTalk.id }}){{ regularTalk.title}} 

{{ regularTalk.description }}

    {% assign events = "" | split: "" %}
    {% for event in site.data.events %}
        {% for appearance in event.appearances %}
            {% if appearance.id == regularTalk.id %}
                {% assign events = events | push: event %}
            {% endif %}
        {% endfor %}
    {% endfor %}

    {% if events.size > 0 %}
### Appearances ({{events.size}})
<ul>
    {% for event in events %}
    <li><a href="/#{{event.id}}">{{event.name}} {{event.year}}</a></li>
    {% endfor %}
</ul>
    {% endif %}

{% endfor %}