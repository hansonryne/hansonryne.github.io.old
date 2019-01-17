---
layout: page
show_meta: false
title: "Travel/Lifestyle"
subheadline: "Blogging about the travel lifestyle.  Tips and lessons from the road."
header:
   image_fullwidth: "header-bus.jpg"
permalink: "/travel-lifestyle/"
---
<ul>
    {% for post in site.categories.travel-lifestyle %}
    <li><a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>
