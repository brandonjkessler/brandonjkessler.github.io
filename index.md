---
title: Home
permalink: "index.html"
---

<nav id="main-nav">
<ul>
    <li><a href="index.html">Home</a></li>
    <li><a href="donate.html">Donate</a></li>
</ul>
</nav>

## Posts

<div>
  {% for post in site.posts %}
    <div>
      {{ post.excerpt }}
      <a href="{{ post.url }}" target="_blank">Read More</a>
      <hr>
    </div>
  {% endfor %}
</div>
