---
title: "Archived Posts"
permalink: /archive/
layout: archive
author_profile: false
search: false
sitemap: false
---

These older posts are preserved for archival purposes and are intentionally left out of the main browsing views.

{% assign archived_posts = site.posts | where_exp: "post", "post.hidden == true" | group_by_exp: "post", "post.date | date: '%Y'" %}
{% assign entries_layout = page.entries_layout | default: "list" %}

{% for year in archived_posts %}
  <section class="taxonomy__section">
    <h2 class="archive__subtitle">{{ year.name }}</h2>
    <div class="entries-{{ entries_layout }}">
      {% for post in year.items %}
        {% include archive-single.html type=entries_layout %}
      {% endfor %}
    </div>
  </section>
{% endfor %}
