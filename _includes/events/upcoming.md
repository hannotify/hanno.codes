
{% capture nowUnix %}{{'now' | date: '%s'}}{% endcapture %}
{% assign upcomingEvents = "" | split: "" %}

{% for event in site.data.events %}
    {% if event.dateEnd %}
        {% capture eventTime %}{{event.dateEnd | date: '%s'}}{% endcapture %}
    {% else %}
        {% capture eventTime %}{{event.dateStart | date: '%s'}}{% endcapture %}
    {% endif %}
    
    {% if nowUnix < eventTime or eventTime == '' %}
        {% assign upcomingEvents = upcomingEvents | push: event | sort: 'dateStart' %}
    {% endif %}
{% endfor %}

{% if upcomingEvents != null and upcomingEvents.size > 0 %}
# [](#upcoming-events)Upcoming ({{upcomingEvents.size}})
{% endif %}

{% for event in upcomingEvents %}
    {% if event.dateEnd %}
        {% capture dateString %}{{ event.dateStart | date: "%B %e, %Y" }} - {{ event.dateEnd | date: "%B %e, %Y" }}{% endcapture %}
    {% elsif event.dateStart %}
        {% assign dateString = event.dateStart | date: "%B %e, %Y" %}
    {% else %}
        {% assign dateString = "To be announced" %}
    {% endif %}


    {% if event.city %}
        {% capture locationString %}{{event.city}}, {{event.country.name}} :{{event.country.flag}}:{% endcapture %}
    {% else %}
        {% assign locationString = "Online" %}
    {% endif %}

    {% include events/partial/event.html %}
{% endfor %}