---
title: "Dev Notes"
layout: archive
permalink: /dev-notes/
author_profile: false
---

{% assign posts = site.categories['dev-notes'] %}
{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
