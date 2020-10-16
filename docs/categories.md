---
layout: default
title: Categories
description: We've got something for just about everyone
---

# All Categories
{% assign sorted_cats = site.categories | sort %}
{% for category in sorted_cats %}{% capture category_name %}{{ category | first }}{% endcapture %}[{{ category_name }}]({{ site.baseurl }}/categories.html#{{ category_name }}) - {% endfor %}

{% assign sorted_cats = site.categories | sort %}
{% for category in sorted_cats %}
{% capture category_name %}{{ category | first }}{% endcapture %}
# {{ category_name }}<a name="{{ category_name }}"></a>
{% for post in site.categories[category_name] %}
### [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
#### {{ post.date | date_to_long_string }}
##### *{{ post.description }}*
<br>
{% endfor %}
{% endfor %}
