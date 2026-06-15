---
title: "Dev Notes / AWS"
layout: archive
permalink: /dev-notes/aws/
author_profile: true
---

{% assign posts = site.categories['aws'] %}
{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
