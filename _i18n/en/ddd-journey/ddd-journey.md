## DDD Logbook: From Start to Finish

Welcome to this page dedicated to my **Domain-Driven Design (DDD) logbook**. Here you will find a series of articles that trace my journey of learning and applying DDD, from the very first concept to the most advanced approaches. Dive with me into this adventure, discover my experiences, challenges, and successes, and let's explore the depths of DDD together.

--- 
<br>
<ul class="post-list">
    {% for post in site.posts %}
        {% if post.tags contains 'ddd-journey' %}
            <li class="post-item">
            <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
            <p class="post-meta">Post at {{ post.date | date: "%d %B %Y" }}</p>
            <p class="post-excerpt">
                {{ post.excerpt }}
            </p>
            <a class="read-more" href="{{ post.url }}">Read &rarr;</a>
            </li>
        {% endif %}
    {% endfor %}
</ul>