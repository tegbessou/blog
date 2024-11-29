![Bienvenue sur mon blog](/assets/images/homepage.jpg)

## Bienvenue sur mon blog dédié au développement et à l'architecture logicielle !

Je suis **Hugues Gobet**, développeur passionné par le **TDD**, le **DDD** et les bonnes pratiques de programmation. Ici, je partage mes projets, mes expériences et mes découvertes pour vous inspirer et échanger autour du code.

<br><br>

### Dernières sorties

---
<br>
<ul class="post-list">
  {% for post in site.posts %}
    <li class="post-item">
      <h2><a href="/fr{{ post.permalink }}">{{ post.title }}</a></h2>
      <p class="post-meta">Publié le {{ post.date | date: "%d %B %Y" }}</p>
      <p class="post-excerpt">
        {{ post.resume }}
      </p>
      <a class="read-more" href="/fr{{ post.permalink }}">Lire la suite &rarr;</a>
    </li>
  {% endfor %}
</ul>