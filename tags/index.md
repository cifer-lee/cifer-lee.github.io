---
layout: default
title: TAGS
---

<div class="tags">
    <h1>Tags</h1>
    {% for tag in site.tags %}
    <a style="font-size: {{ tag | last | size | times: 100 | divided_by: site.tags.size | plus: 80 }}%;" href="/tagposts.html" title="">{{ tag| first }}</a>
    {% endfor %}
</div>
