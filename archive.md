---
layout: default
title: Archive
---

# Archive

Browse all posts by month and year.

{% assign postsByYearMonth = site.categories["blog"] | group_by_exp: "post", "post.date | date: '%B %Y'" %}
{% for yearMonth in postsByYearMonth %}
  
  <h2>{{ yearMonth.name }}</h2>
  <ul>
    {% for post in yearMonth.items %}
      {% unless post.categories[0] == "notes" %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {% endunless %}
    {% endfor %}
  </ul>
{% endfor %}
