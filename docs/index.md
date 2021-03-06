---
layout: default
title: David Shapiro's Tech Blog
---

# About Me

I am a professional IT engineer by trade, focusing on private cloud infrastructure technologies such as VMware, Microsoft, and SAN storage. I have a strong focus on scripting and automation with Python, PowerShell, and Rundeck. 

Beyond that, my hobbies and interests include science fiction writing, tinkering with deep learning, quantum computing, and the Singularity in general. I also enjoy spending time in nature and biking. 

---
# Blog Posts

{% for post in site.posts %}

### [{{ post.title }}]({{ site.baseurl }}{{ post.url }})
#### {{ post.date | date_to_long_string }}
##### *{{ post.description }}*
<br>

{% endfor %}
