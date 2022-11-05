{% assign articlesByYear = site.data.articles | group_by: "year" %}

This is a list of articles that I (co-)wrote, but were published externally.

{% for articles in articlesByYear %}

# {{ articles.name }}

    {% for article in articles.items %} 

## [](#{{ article.title | slugify }}){{ article.title}}

{{article.synopsis}}

**Language:** :{{article.language}}:<br/>
**Source:** [{{article.source}}]({{article.url}})

    {% endfor %}
{% endfor %}