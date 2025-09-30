{% if include.talks.size > 0 %}
# [](#{{ include.anchor }}){{ include.title }}
{% endif %}

{{ include.description }}

{% for talk in include.talks %}

{% if talk.retired %}
<div class="notice" markdown="1">
{% endif %}

## {{ talk.title }} {#{{talk.id}}}

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
<div class="notice--info">
    <strong>Appearances ({{events.size}})</strong>
    <ul>
        {% for event in events %}
        <li><a href="/events#{{event.id}}">{{event.name}} {{event.year}}</a></li>
        {% endfor %}
    </ul>
</div>
    {% endif %}

{% if talk.retired %}
</div>
{% endif %}

{% endfor %}
