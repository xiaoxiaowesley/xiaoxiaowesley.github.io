---
layout: page
title: Blog Archive
---

{% for tag in site.tags %}
  <h3>{{ tag[0] }}</h3>
  <ul>
    {% for post in tag[1] %}   


    <a href="{{ post.url }}">
        <li>
          <div>
            {% assign foundImage = 0 %}
              {% assign images = post.content | split:"<img " %}
                  {% for image in images %}
                    {% if image contains 'src' %}
                        {% if foundImage == 0 %}
                          {% assign html = image | split:"/>" | first %}
                            <time>{{ post.date | date:"%d %b %Y" }}</time>
                            <h3>{{ post.title }}</h3>
                            <hr>
                            <div><img width="250" {{ html }} />{{ post.desc }}</div>
                          {% assign foundImage = 1 %}
                        {% endif %}
                        {% endif %}
                {% endfor %}
            </div>
        </li>
    </a>


    {% endfor %}
  </ul>
{% endfor %}