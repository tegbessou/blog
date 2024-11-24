![Bienvenue sur mon blog](/assets/images/homepage.jpg)

## Welcome to my blog dedicated to development and software architecture!

I am **Hugues Gobet**, a developer passionate about **TDD**, **DDD**, and programming best practices. Here, I share my projects, experiences, and discoveries to inspire you and engage in discussions around code.

<br><br>

### Latest Posts (coming soon)

---
<br>
<ul class="post-list">
  {% for post in site.posts %}
    <li class="post-item">
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      <p class="post-meta">Publié le {{ post.date | date: "%d %B %Y" }}</p>
      <p class="post-excerpt">
        {{ post.excerpt }}
      </p>
      <a class="read-more" href="{{ post.url }}">Lire la suite &rarr;</a>
    </li>
  {% endfor %}
</ul>