---
layout: blog
permalink: /blog/
title: "Blog"
---

<section class="blog-hero">
  <h1>Welcome to Jiawei's Log</h1>
  <p>
    Notes on AI4Science, protein modeling, research engineering, and the small lessons that only become obvious after writing them down.
  </p>
  <div class="blog-socials" aria-label="Profile links">
    <a href="https://github.com/JiaweiZhang1997" aria-label="GitHub"><i class="fab fa-github"></i></a>
    <a href="https://scholar.google.com/citations?user=-OmES9gAAAAJ" aria-label="Google Scholar"><i class="ai ai-google-scholar"></i></a>
    <a href="mailto:jiawei_zhang1@163.com" aria-label="Email"><i class="fas fa-envelope"></i></a>
    <a href="{{ '/feed.xml' | relative_url }}" target="_self" aria-label="RSS"><i class="fas fa-rss"></i></a>
  </div>
</section>

<section class="blog-post-list" aria-label="Recent posts">
  {% for post in site.posts %}
    <article class="blog-card">
      <a href="{{ post.url | relative_url }}" target="_self">
        <h2>{{ post.title }}</h2>
        <p>{{ post.excerpt | strip_html | truncate: 190 }}</p>
        <span class="blog-post-meta">
          Date: {{ post.date | date: "%B %-d, %Y" }}
          {% if post.reading_time %} | Estimated Reading Time: {{ post.reading_time }}{% endif %}
          {% if post.author %} | Author: {{ post.author }}{% endif %}
        </span>
      </a>
    </article>
  {% else %}
    <p class="blog-empty">No posts yet.</p>
  {% endfor %}
</section>
