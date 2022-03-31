# Brandon J. Kessler

Migrating from a full WordPress site to a static site. [BrandonJKessler.com](https://www.brandonjkessler.com/) will be moving here.


## Posts


{% for post in site.posts %}
    {% if post.excerpt %}
        ### {{ post.title }}
        {{ post.excerpt }}
    {% endif %}
{% endfor %}
