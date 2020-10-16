---
layout: default
title: Categories
---

{% for category in site.categories %}
{% capture category_name %}{{ category | first }}{% endcapture %}
# {{ category_name }}

{% for post in site.categories[category_name] %}
- [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
  - {{ post.date | date_to_long_string }}
  - *{{ post.description }}*
{% endfor %}

{% endfor %}
