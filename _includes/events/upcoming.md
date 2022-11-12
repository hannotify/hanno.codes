
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
        {% capture locationString %}{{event.city}}, {{event.country.name}} <img src="images/flags/{{event.country.flag}}.gif"/>{% endcapture %}
    {% else %}
        {% assign locationString = "Online" %}
    {% endif %}

## [](#{{event.id}})[{{event.name}} {{event.year}}]({{event.url}})

<table>
    <tr>
        <td><strong>Location</strong></td>
        <td>{{locationString}}</td>
    </tr>
    <tr>
        <td><strong>Date</strong></td>
        <td>{{dateString}}</td>
    </tr>
    {% if event.occasion %}
    <tr>
        <td><strong>Occasion</strong></td>
        <td>{{event.occasion}}</td>
    </tr>
    {% endif %}
    {% if event.appearances.size > 0 %}
        {% for appearance in event.appearances %}
            {% assign talk = site.data.talks | where: "id", appearance.id | first %}

    <tr>
        <td><strong>Talk</strong></td>
        <td><a href="talks#{{talk.id}}">{{talk.title}}</a></td>
    </tr>

        {% if talk.cospeakers != null and talk.cospeakers.size > 0 %}
            {% for cospeaker in talk.cospeakers %}
            <tr>
                <td style="text-align: right">Co-speaker</td>
                <td><a href="{{cospeaker.twitterUrl}}" target="_blank">{{cospeaker.name}}</a></td>
            </tr>
            {% endfor %}
        {% endif %}
        {% if appearance.cospeakers != null and appearance.cospeakers.size > 0 %}
            {% for cospeaker in appearance.cospeakers %}
            <tr>
                <td style="text-align: right">Co-speaker</td>
                <td><a href="{{cospeaker.twitterUrl}}" target="_blank">{{cospeaker.name}}</a></td>
            </tr>
            {% endfor %}
        {% endif %}
        
        {% if appearance.sessionPageUrl%}
            <tr>
                <td>&nbsp;</td>
                <td>
                    <a target="_blank" href="{{appearance.sessionPageUrl}}"><code>session page</code></a>
                </td>
            </tr>
        {% endif %}
        {% endfor %}
    {% else %}
    <tr>
        <td><strong>Talk</strong></td>
        <td>TBD</td>
    </tr>
    {% endif %}
</table>
{% endfor %}