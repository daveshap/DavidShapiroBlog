---
layout: default
title: Categories
---

# All Categories
{% for category in site.categories | sort %}
{% capture category_name %}{{ category | first }}{% endcapture %}
- {{ category_name }}
{% endfor %}

# Posts by Category
{% for category in site.categories | sort %}
{% capture category_name %}{{ category | first }}{% endcapture %}
## {{ category_name }}
{% for post in site.categories[category_name] %}
### [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
#### {{ post.date | date_to_long_string }}
##### *{{ post.description }}*
<br>
{% endfor %}
{% endfor %}
