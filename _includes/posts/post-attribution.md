<p>
    Posted on {{ post.date | date_to_string }} by <strong>{{ post.author }}</strong>
    {% for label in post.labels %}
        <a href="#"><code>{{ label }}</code></a>
    {% endfor %}
</p>