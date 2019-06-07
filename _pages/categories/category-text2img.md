---
title: "Post about Text2img"
layout: archive
permalink: /categories/text2img
author_profile: true
---

{% assign posts = site.categories.text2img | sort:"date" %}

{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
