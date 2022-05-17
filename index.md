<nav>
{% assign pages = site.pages %}
{% for page in pages %}
  <li><a href="{{page.url}}">{{page.title}}</a></li>
{% endfor %}
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
