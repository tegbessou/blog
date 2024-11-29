### Special Articles

This page brings together special articles related to software architecture that are not part of the logbook. Here you'll find a variety of topics and in-depth analyses to enhance your understanding of software development.

--- 
<br>
<ul class="post-list">
    {% for post in site.posts %}
        {% if post.categories contains 'other' %}
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