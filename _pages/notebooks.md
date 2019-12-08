---
layout: archive
title: "Notebooks"
permalink: /notebooks/
author_profile: true
redirect_from: 
  - /Notebook/
---

{% include base_path %}

{% for post in site.notebooks reversed %}
{% if post.url contains "python" %}
Python:
* [{{post.title}}]({{ post.url }})
{% endif %}
{% endfor %}
