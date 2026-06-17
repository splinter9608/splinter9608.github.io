---
title: "Dev Notes / Blog Setup"
layout: archive
permalink: /dev-notes/blog-setup/
author_profile: false
---

{% assign posts = site.categories['blog-setup'] %}
{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
