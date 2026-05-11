---
layout: blog
permalink: /blog/tags/
title: "Tags"
---

<section class="blog-section-head">
  <h1>Tags</h1>
  <p>Browse notes by recurring themes.</p>
</section>

<section class="blog-tags">
  {% assign sorted_tags = site.tags | sort %}
  {% for tag in sorted_tags %}
    <h2 id="{{ tag[0] | slugify }}">{{ tag[0] }}</h2>
    <ol>
      {% for post in tag[1] %}
        <li>
          <a href="{{ post.url | relative_url }}" target="_self">{{ post.title }}</a>
          <span>{{ post.date | date: "%B %-d, %Y" }}</span>
        </li>
      {% endfor %}
    </ol>
  {% endfor %}
</section>
