Migrating from a full WordPress site to a static site. [BrandonJKessler.com](https://www.brandonjkessler.com/) will be moving here.


## Posts

{% for post in site.posts %}
    <a href="{{ post.url }}"><h3>{{ post.title }}</h3></a>
    {% if post.excerpt %}
        {{ post.excerpt }}
    {% endif %}
{% endfor %}
