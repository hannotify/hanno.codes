---
layout: default
title: Blog
---

# Blog

{% for post in site.posts %}

## [{{post.title}}]({{post.url}})

<p>{{ post.date | date_to_string }} - {{ post.author }}</p>

{{ post.excerpt }}

{% endfor %}