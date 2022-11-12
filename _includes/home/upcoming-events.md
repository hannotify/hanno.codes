
{% capture nowUnix %}{{'now' | date: '%s'}}{% endcapture %}
{% assign upcomingEvents = "" | split: "" %}

{% for event in site.data.events | sort: 'date' | reverse %}
    {% if event.dateEnd %}
        {% capture eventTime %}{{event.dateEnd | date: '%s'}}{% endcapture %}
    {% else %}
        {% capture eventTime %}{{event.dateStart | date: '%s'}}{% endcapture %}
    {% endif %}
    
    {% if nowUnix < eventTime or eventTime == '' %}
        {% assign upcomingEvents = upcomingEvents | push: event %}
    {% endif %}
{% endfor %}

{% for event in upcomingEvents %}

[![{{event.name}} {{event.year}}](../../assets/images/conference-logos/{{event.logo}})](events#{{event.id}})

{% endfor %}