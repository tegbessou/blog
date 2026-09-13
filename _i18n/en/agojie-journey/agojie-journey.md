## Agojie Logbook

This page brings together the series on developing **Agojie**, which automates the board game **Dice Throne**, from the first commit to the end. Each episode tells one step of the project and the modeling decisions that come with it.

--- 
<br>
<ul class="post-list">
    {% for post in site.posts %}
        {% if post.categories contains 'agojie-logbook' %}
            <li class="post-item">
            <h2><a href="{{ post.permalink }}">{{ post.title }}</a></h2>
            <p class="post-meta">Post at {{ post.date | date: "%d %B %Y" }}</p>
            <p class="post-excerpt">
                {{ post.resume }}
            </p>
            <a class="read-more" href="{{ post.permalink }}">Read &rarr;</a>
            </li>
        {% endif %}
    {% endfor %}
</ul>
