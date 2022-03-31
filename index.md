Migrating from a full WordPress site to a static site. [BrandonJKessler.com](https://www.brandonjkessler.com/) will be moving here.


## Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
