---
title: "Post about notes"
layout: archive
permalink: /categories/notes
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.notes | sort:"date" %}

{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
