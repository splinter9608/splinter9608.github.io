---
title: "Dev Notes"
layout: archive
permalink: /categories/dev-notes/
author_profile: true
---

개발 환경 구축, 도구 설정, 프로젝트 개발 기록을 남깁니다.

---

## 카테고리

- [🤖 AI 환경 구축](/dev-notes/ai-setup/) — Claude, MCP, NotebookLM 등 AI 툴 세팅
- [🖥️ VSCode](/dev-notes/vscode/) — 익스텐션, 설정, 단축키
- [☁️ AWS](/dev-notes/aws/) — 인프라 구축 및 실습
- [📦 프로젝트](/dev-notes/project/) — 프로젝트별 개발 기록

---

{% assign posts = site.categories['dev-notes'] %}
{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
