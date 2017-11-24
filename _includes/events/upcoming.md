# [](#upcoming-events)Upcoming events

{% capture nowUnix %}{{'now' | date: '%s'}}{% endcapture %}

{% for event in site.data.events %}
    {% capture eventTime %}{{event.dateStart | date: '%s'}}{% endcapture %}

    {% if nowUnix < eventTime %}
        {% if event.dateEnd %}
            {% capture dateString %}{{ event.dateStart | date: "%B %e, %Y" }} - {{ event.dateEnd | date: "%B %e, %Y" }}{% endcapture %}
        {% else %}
            {% assign dateString = event.dateStart | date: "%B %e, %Y" %}
        {% endif %}
## [](#{{event.id}})[{{event.name}} {{event.year}}]({{event.url}})

|-------------:|:----------------------------------------|
| **Location** | {{event.country}}, {{event.city}}       |
|     **Date** | {{dateString}}                          |
        {% for appearance in event.appearances %}{% assign talk = site.data.talks | where: "id", appearance.id | first %}|     **Talk** | [{{talk.title}}](talks#{{talk.id}})     |
        {% endfor %}
    {% endif %}
{% endfor %}