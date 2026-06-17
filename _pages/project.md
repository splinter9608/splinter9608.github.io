---
title: "Project"
layout: archive
permalink: /project/
author_profile: false
---

{% assign posts = site.categories['project'] %}
{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
