{% assign posts = site.posts %}

{% assign entries_layout = page.entries_layout | default: 'list' %}
<div class="entries-{{ entries_layout }}">
  {% for post in posts limit: 4 %}
    {% include archive-single.html type=entries_layout %}
  {% endfor %}
</div>
