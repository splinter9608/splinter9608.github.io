---
title: "Work Log"
layout: archive
permalink: /categories/work-log/
author_profile: true
---

{% assign posts = site.categories['work-log'] %}
{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
