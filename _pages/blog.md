---
layout: blog
permalink: /blog/
title: "Blog"
---

<section class="blog-hero">
  <div>
    <p class="blog-kicker">Research notes and engineering logs</p>
    <h1>Jiawei's Log</h1>
    <p>
      A compact writing space for AI4Science notes, protein modeling experiments, paper readings, and the engineering decisions that are worth keeping.
    </p>
  </div>
  <div class="blog-hero-panel" aria-label="Blog summary">
    <span>{{ site.posts | size }}</span>
    <p>published notes</p>
    <a href="{{ '/feed.xml' | relative_url }}" target="_self">RSS feed</a>
  </div>
</section>

<section class="blog-topics" aria-label="Topics">
  <a href="{{ '/blog/tags/#ai4science' | relative_url }}" target="_self">AI4Science</a>
  <a href="{{ '/blog/tags/#protein-modeling' | relative_url }}" target="_self">Protein Modeling</a>
  <a href="{{ '/blog/tags/#engineering' | relative_url }}" target="_self">Engineering</a>
  <a href="{{ '/blog/tags/#notes' | relative_url }}" target="_self">Notes</a>
</section>

<section class="blog-section-head blog-section-head--posts">
  <h2>Latest Posts</h2>
</section>

<section class="blog-post-list" aria-label="Recent posts">
  {% for post in site.posts %}
    <article class="blog-card">
      <a href="{{ post.url | relative_url }}" target="_self">
        {% if post.tags %}
          <div class="blog-card__tags">
            {% for tag in post.tags limit: 3 %}
              <span>{{ tag }}</span>
            {% endfor %}
          </div>
        {% endif %}
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
