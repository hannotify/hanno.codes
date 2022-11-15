
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

# [](#{{eventByYear.name}}){{eventByYear.name}} ({{eventByYear.items.size}})

    {% for event in eventByYear.items %}
    {% assign year = eventByYear.name | plus: 0 %}
    {% include events/event.html %}
    {% endfor %}
{% endfor %}
