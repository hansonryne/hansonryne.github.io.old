---
layout: page
show_meta: false
title: "Travel"
subheadline: "Blogging about the travel lifestyle.  Tips and lessons from the road."
header:
   image_fullwidth: "header-bus.jpg"
permalink: "/travel/"
---
<ul>
    {% for post in site.categories.travel %}
    <li><a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>
