---
layout: archive
title: "Categorie"
permalink: /categories/
---

{% assign category_map = 
  "ai:Intelligenza Artificiale|
   data:Data|
   postgresql:PostgreSQL|
   sql server:SQL Server" | split: "|" %}

{% assign category_dict = "" | split: "" %}

{% for item in category_map %}
  {% assign parts = item | split: ":" %}
  {% assign key = parts[0] | strip %}
  {% assign value = parts[1] | strip %}
  {% assign category_dict = category_dict | push: key | push: value %}
{% endfor %}

{% for category in site.categories %}
  {% assign cat_name = category[0] %}
  {% assign display_name = cat_name %}

  {% for item in category_map %}
    {% assign parts = item | split: ":" %}
    {% if parts[0] == cat_name %}
      {% assign display_name = parts[1] %}
    {% endif %}
  {% endfor %}

  <h2>{{ display_name }} ({{ category[1].size }})</h2>

  {% for post in category[1] %}
    <p><a href="{{ post.url }}">{{ post.title }}</a></p>
  {% endfor %}
{% endfor %}
