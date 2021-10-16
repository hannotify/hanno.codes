<p>
    Posted on {{ post.date | date_to_string }} by <strong>{{ post.author }}</strong>
    {% for tag in post.tags %}
        <a href="/tags/{{ tag }}"><code>{{ tag }}</code></a>
    {% endfor %}
</p>