---
title: "Dev Notes / 프로젝트"
layout: archive
permalink: /dev-notes/project/
author_profile: true
---

{% assign posts = site.categories['project'] %}
{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
