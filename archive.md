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
                            <div style="display: flex; align-items: center; justify-content: space-between; border:1px solid #ddd; border-radius:10px; padding:20px; background-color: #fff; box-shadow:04px6px rgba(0,0,0,0.1);">
                              <div style="display: flex; flex-direction: column; padding:20px; flex-basis:50%;">
                                <div style="font-size:24px; font-weight: bold; color: #333; margin-bottom:10px;">{{ post.title }}</div>
                                <div style="font-size:16px; color: #999; margin-top: auto;">{{ post.date | date:"%d %b %Y" }}</div>
                              </div>
                              <div style="display: flex; align-items: center; margin-left:20px;">
                               <div><img style="display: block; width:100px; height:100px; object-fit: cover; border-radius:50%; box-shadow:04px6px rgba(0,0,0,0.1);" {{ html }} />{{ post.desc }}</div>
                              </div>
                            </div>
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