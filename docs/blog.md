---
layout: default
title: Latest Blog Posts
---

{% for post in site.posts %}

## [{{ post.title }}]({{ site.baseurl }}{{ post.url }})

### {{ post.date | date_to_long_string }}

*{{ post.description }}*

{% endfor %}
