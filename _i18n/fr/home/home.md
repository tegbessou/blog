![Bienvenue sur mon blog](/assets/images/homepage.jpg)

## Bienvenue sur mon blog dédié au développement et à l'architecture logicielle !

Je suis **Hugues Gobet**, développeur passionné par le **TDD**, le **DDD** et les bonnes pratiques de programmation. Ici, je partage mes projets, mes expériences et mes découvertes pour vous inspirer et échanger autour du code.

<br><br>

### Dernières sorties (arrive bientôt)

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