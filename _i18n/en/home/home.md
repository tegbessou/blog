![Bienvenue sur mon blog](/assets/images/homepage.jpg)

## Welcome to my blog dedicated to development and software architecture!

I am **Hugues Gobet**, a developer passionate about **TDD**, **DDD**, and programming best practices. Here, I share my projects, experiences, and discoveries to inspire you and engage in discussions around code.

<br><br>

### Latest Posts

---
<br>
<ul class="post-list">
  {% for post in site.posts %}
    <li class="post-item">
      <h2><a href="{{ post.permalink }}">{{ post.title }}</a></h2>
      <p class="post-meta">Publish at {{ post.date | date: "%d %B %Y" }}</p>
      <p class="post-excerpt">
        {{ post.resume }}
      </p>
      <a class="read-more" href="{{ post.permalink }}">Read &rarr;</a>
    </li>
  {% endfor %}
</ul>