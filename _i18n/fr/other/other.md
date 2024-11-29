### Articles Hors Série

Cette page rassemble des articles spéciaux en rapport avec l'architecture logicielle, qui ne font pas partie du journal de bord. Vous y découvrirez des sujets variés et des analyses approfondies pour enrichir votre compréhension du développement logiciel.

--- 
<br>
<ul class="post-list">
    {% for post in site.posts %}
        {% if post.categories contains 'other' %}
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
