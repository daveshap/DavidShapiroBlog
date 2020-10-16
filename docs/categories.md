---
layout: default
title: Categories
---

{% for category in site.categories %}
{% capture category_name %}{{ category | first }}{% endcapture %}
# {{ category_name }}

{% for post in site.categories[category_name] %}
* {{ post.date | date_to_long_string }} - [{{ post.title }}]({{ site.baseurl }}{{ post.url }}) - *{{ post.description }}*
{% endfor %}

{% endfor %}
