---
layout: blog
permalink: /blog/archive/
title: "Archive"
---

<section class="blog-section-head">
  <h1>Archive</h1>
  <p>All posts grouped by publication date.</p>
</section>

<section class="blog-archive">
  {% assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
  {% for year in posts_by_year %}
    <h2>{{ year.name }}</h2>
    <ol>
      {% for post in year.items %}
        <li>
          <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%b %-d" }}</time>
          <a href="{{ post.url | relative_url }}" target="_self">{{ post.title }}</a>
        </li>
      {% endfor %}
    </ol>
  {% endfor %}
</section>
