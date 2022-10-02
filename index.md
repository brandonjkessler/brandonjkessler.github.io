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

# Posts

<div>
  {% for post in site.posts %}
    <div>
      <hr>
      <p>Begin Post</p>
      <hr>
      {{ post.excerpt }}
      <a href="{{ post.url }}" target="_blank">Read More</a>
      <hr>
      <p>End Post</p>
      <hr>
    </div>
  {% endfor %}
</div>
