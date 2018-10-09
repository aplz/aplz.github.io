---
layout: default
title: Insights and musings on solving real world problems with machine learning.
---

{% for post in site.posts %}   
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <p><small><strong>{{ post.date | date: "%B %e, %Y" }}</strong></small></p>            
{% endfor %}