---
layout: page
title: Blog Archive
---

{% for tag in site.tags %}
  <h3>{{ tag[0] }}</h3>
  <ul>
    {% for post in tag[1] %}
   
      {% assign matches = post.content | scan: /![.*?]((.*?))/ %}
      {% if matches.size >0 %}
      {% assign image_url = matches[0][0] %}
      {{ image_url }}
      {% else %}
        No image found in the post
      {% endif %}

      <li><a href="{{ post.url }}">{{ post.date | date: "%B %Y" }} - {{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}