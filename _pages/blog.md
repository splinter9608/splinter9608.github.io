---
title: "Blog"
layout: archive
permalink: /blog/
author_profile: false
---

{% assign posts = site.categories['blog'] %}
{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
