---
layout: archive
title: "Notebooks"
permalink: /notebooks/
author_profile: true
---

{% include base_path %}

{% for post in site.notebooks reversed %}
{% assign url = post.url |  split: '/'%}
{{url[2] | capitalize}}:
{% for post in site.notebooks reversed %}
{% if post.url contains url[2] %}
* [{{post.title}}]({{ post.url }})
{% endif %}
{% endfor %}
{% endfor %}

