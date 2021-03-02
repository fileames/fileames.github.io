---
layout: default
title: notes
---

## Notes

Notes taken to keep track of the topics I learn. 
I try to add references but some of the old notes taken before this website may not have references, sorry about that.

<div id="archives">
{% for category in site.categories %}
    {% capture category_name %}{{ category | first }}{% endcapture %}
    {% unless category_name == "notes" or category_name == "blog" %}
    <div class="archive-group">
        <h3 class="category-head">{{ category_name }}</h3>
        <a name="{{ category_name | slugize }}"></a>
        <ul>
        {% for post in site.categories[category_name] %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endfor %}
        </ul>
    </div>
    {% endunless %} 
{% endfor %}
</div>

