---
layout: default
---

# Upcoming events

{% include events/upcoming.md %}

[More events ➡️](/events)

# Latest blog posts

{% for post in site.posts limit:3 %}
{% include posts/post-overview.md %}
{% endfor %}

[More blog posts ➡️](/blog)

# hanno.codes?

[hanno.codes](https://hanno.codes) is the personal website of Hanno Embregts. 

{% include bio/short-bio.md %}

[More about me ➡️](/bio)

