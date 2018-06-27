---
title: Categories
permalink: /categories/
---

{% for category in site.categories %}
  <h4>{{ category[0] }}</h4>

  <ul class="post-list">

  {% for posts in category %}
    {% for post in posts %}
      {% if post.title != nil %}

        <!-- <li><a href="{{ post.url }}">{{ post.title }}</a></li> -->

        <li>
          {% assign date_format = site.minima.date_format | default: "%b %-d, %Y" %}
          <span class="post-meta">{{ post.date | date: date_format }}</span>

          <h2>
            <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
          </h2>
        </li>


      {% endif %}
    {% endfor %}
  {% endfor %}
  </ul>
{% endfor %}