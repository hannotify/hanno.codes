---
layout: single
title: Articles
author_profile: false
header:
  image: /assets/images/overlay/devoxx-uk-2022.jpg
  caption: "Devoxx UK 2022"
permalink: /articles/
toc: true
toc_sticky: true
toc_label: Articles
---

{% assign articlesByYear = site.data.articles | group_by: "year" %}

This is a list of articles that I (co-)wrote, but were published externally.

{% for articles in articlesByYear %}

# {{ articles.name }}

    {% for article in articles.items %} 

## [](#{{ article.title | slugify }}){{ article.title}}

{{article.synopsis}}

**Language:** :{{article.language}}:<br/>
**Source:** [{{article.source}}]({{article.url}}){:target="_blank"}

    {% endfor %}
{% endfor %}
