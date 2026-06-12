---
title: "Dev Notes"
layout: archive
permalink: /categories/dev-notes/
author_profile: true
---

{% assign posts = site.categories['dev-notes'] %}
{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
