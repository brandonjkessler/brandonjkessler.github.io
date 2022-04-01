Migrating from a full WordPress site to a static site. [BrandonJKessler.com](https://www.brandonjkessler.com/) will be moving here.


## Posts

<ul>
  {% for post in site.posts %}
    <li>
      {{ post.excerpt }}
      <a href="{{ post.url }}">Read More</a>
      <hr>
    </li>
  {% endfor %}
</ul>
