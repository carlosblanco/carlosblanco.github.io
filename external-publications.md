---
layout: page
title: External Publications
permalink: /publications/
---

Here is a collection of articles and tutorials I have originally published on other company blogs and platforms.

<ul>
  {% for post in site.posts %}
    {% if post.original_site %}
      <li>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a> 
        <span style="color: #666; font-size: 0.9em;">(via {{ post.original_site }})</span>
      </li>
    {% endif %}
  {% endfor %}
</ul>
