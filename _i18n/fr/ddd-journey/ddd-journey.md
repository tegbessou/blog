## Journal de bord du DDD : Du début à la fin

Bienvenue sur cette page dédiée à mon **journal de bord du Domain-Driven Design (DDD)**. Vous y trouverez une série d'articles qui retracent mon parcours d'apprentissage et de mise en pratique du DDD, du tout premier concept aux approches les plus avancées. Plongez avec moi dans cette aventure, découvrez mes expériences, mes défis et mes réussites, et explorons ensemble les profondeurs du DDD.

--- 
<br>
<ul class="post-list">
    {% for post in site.posts %}
        {% if post.categories contains 'ddd-logbook' %}
            <li class="post-item">
            <h2><a href="/fr/{{ post.permalink }}">{{ post.title }}</a></h2>
            <p class="post-meta">Publié le {{ post.date | date: "%d %B %Y" }}</p>
            <p class="post-excerpt">
                {{ post.resume }}
            </p>
            <a class="read-more" href="/fr/{{ post.permalink }}">Lire la suite &rarr;</a>
            </li>
        {% endif %}
    {% endfor %}
</ul>