{% assign retiredTalks = site.data.talks | where: "retired", "true" %}
{% include talks/talkType.md anchor="retired" title="Retired talks" talks=retiredTalks description="Retired talks are talks that have been presented by me in the past, but at some point I've stopped presenting them. In most cases this is because the topic has become somewhat outdated. And in some cases it is because I no longer feel comfortable presenting it. I might have better talks available to submit, or I might feel like I'm no longer the best person to talk about a particular subject. 

I'll include my retired talks here for the sake of completeness." %}
