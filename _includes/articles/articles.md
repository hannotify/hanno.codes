{% assign articlesByYear = site.data.articles | group_by: "year" %}

This is a list of articles that I (co-)wrote, but were published externally.

{% for articles in articlesByYear %}

# {{ articles.name }}

    {% for article in articles.items %} 

## [](#{{ article.title | slugify }}){{ article.title}}

{{article.synopsis}}

**Language:** <img src="images/flags/{{article.language}}.gif"/> <br/>
**Source:** [{{article.source}}]({{article.url}})

    {% endfor %}
{% endfor %}