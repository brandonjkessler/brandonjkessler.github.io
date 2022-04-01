Migrating from a full WordPress site to a static site. [BrandonJKessler.com](https://www.brandonjkessler.com/) will be moving here.


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
