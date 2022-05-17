
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
