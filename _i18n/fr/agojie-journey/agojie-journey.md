## Journal de bord d'Agojie

Cette page rassemble la série consacrée au développement d'**Agojie**, l'automatisation du jeu de société **Dice Throne**, du premier commit jusqu'au bout. Chaque épisode raconte une étape du projet et les décisions de modélisation qui l'accompagnent.

--- 
<br>
<ul class="post-list">
    {% for post in site.posts %}
        {% if post.categories contains 'agojie-logbook' %}
            <li class="post-item">
            <h2><a href="/fr{{ post.permalink }}">{{ post.title }}</a></h2>
            <p class="post-meta">Publié le {{ post.date | date: "%d %B %Y" }}</p>
            <p class="post-excerpt">
                {{ post.resume }}
            </p>
            <a class="read-more" href="/fr{{ post.permalink }}">Lire la suite &rarr;</a>
            </li>
        {% endif %}
    {% endfor %}
</ul>
