# [](#{{ include.anchor }}){{ include.title }}

{% assign talks = include.talks %}
{% for talk in talks %}

## [](#{{ talk.id }}){{ talk.title}} 

{{ talk.description }}

    {% assign events = "" | split: "" %}
    {% for event in site.data.events %}
        {% for appearance in event.appearances %}
            {% if appearance.id == talk.id %}
                {% assign events = events | push: event %}
            {% endif %}
        {% endfor %}
    {% endfor %}

    {% if events.size > 0 %}
#### Appearances ({{events.size}})
<ul>
    {% for event in events %}
    <li><a href="/events#{{event.id}}">{{event.name}} {{event.year}}</a></li>
    {% endfor %}
</ul>
    {% endif %}

{% endfor %}