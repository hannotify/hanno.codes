
{% capture nowUnix %}{{'now' | date: '%s'}}{% endcapture %}
{% assign upcomingEvents = "" | split: "" %}

{% for event in site.data.events %}
    {% capture eventTime %}{{event.dateStart | date: '%s'}}{% endcapture %}

    {% if nowUnix < eventTime %}
        {% assign upcomingEvents = upcomingEvents | push: event %}
    {% endif %}
{% endfor %}

{% if upcomingEvents != null and upcomingEvents.size > 0 %}
# [](#upcoming-events)Upcoming events ({{upcomingEvents.size}})
{% endif %}

{% for event in upcomingEvents %}
    {% if event.dateEnd %}
        {% capture dateString %}{{ event.dateStart | date: "%B %e, %Y" }} - {{ event.dateEnd | date: "%B %e, %Y" }}{% endcapture %}
    {% else %}
        {% assign dateString = event.dateStart | date: "%B %e, %Y" %}
    {% endif %}

    {% if event.city %}
        {% capture locationString %}{{event.city}}, {{event.country.name}} <img src="images/flags/{{event.country.flag}}.gif"/>{% endcapture %}
    {% else %}
        {% assign locationString = "Online (webinar)" %}
    {% endif %}

## [](#{{event.id}})[{{event.name}} {{event.year}}]({{event.url}})

|-------------:|:----------------------------------------|
| **Location** | {{locationString}}                      |
|     **Date** | {{dateString}}                          |
    {% for appearance in event.appearances %}{% assign talk = site.data.talks | where: "id", appearance.id | first %}|     **Talk** | [{{talk.title}}](talks#{{talk.id}})     |
    {% endfor %}
{% endfor %}