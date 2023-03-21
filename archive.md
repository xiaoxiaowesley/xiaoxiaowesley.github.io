---
layout: page
title: Blog Archive
---

{% for tag in site.tags %}
  <h3>{{ tag[0] }}</h3>
  <ul>
    {% for post in tag[1] %}
      {% assign images = post.content | markdownify | scan(/![.*]((.*))/) %}
      {% if images.size >0 %}
      <img src="{{ images[0][0] }}" alt="{{ post.title }}">
      {% endif %}
      <li><a href="{{ post.url }}">{{ post.date | date: "%B %Y" }} - {{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}