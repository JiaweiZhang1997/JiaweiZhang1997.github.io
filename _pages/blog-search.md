---
layout: blog
permalink: /blog/search/
title: "Search"
---

<section class="blog-section-head">
  <h1>Search</h1>
  <p>Search post titles, summaries, and tags.</p>
</section>

<section class="blog-search">
  <label class="screen-reader-text" for="blog-search-input">Search posts</label>
  <input id="blog-search-input" type="search" placeholder="Type a keyword..." autocomplete="off">
  <div id="blog-search-results" class="blog-post-list"></div>
</section>

<script>
  var blogPosts = [
    {% for post in site.posts %}
      {
        title: {{ post.title | jsonify }},
        url: {{ post.url | relative_url | jsonify }},
        excerpt: {{ post.excerpt | strip_html | normalize_whitespace | jsonify }},
        date: {{ post.date | date: "%B %-d, %Y" | jsonify }},
        tags: {{ post.tags | join: " " | jsonify }}
      }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ];

  var input = document.getElementById("blog-search-input");
  var results = document.getElementById("blog-search-results");

  function renderSearch(query) {
    var normalized = query.trim().toLowerCase();
    var matches = blogPosts.filter(function(post) {
      var haystack = [post.title, post.excerpt, post.tags].join(" ").toLowerCase();
      return !normalized || haystack.indexOf(normalized) !== -1;
    });

    results.innerHTML = matches.map(function(post) {
      return '<article class="blog-card"><a href="' + post.url + '" target="_self"><h2>' +
        post.title + '</h2><p>' + post.excerpt.substring(0, 190) +
        '</p><span class="blog-post-meta">Date: ' + post.date + '</span></a></article>';
    }).join("") || '<p class="blog-empty">No matching posts.</p>';
  }

  input.addEventListener("input", function(event) {
    renderSearch(event.target.value);
  });

  renderSearch("");
</script>
