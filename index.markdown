---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
title: Accueil
aside:
  toc: true
---

{% for post in site.posts %}
## [{{ post.title }}]({{ post.url | relative_url }})

*{{ post.date | date: "%d %B %Y" }}*

{{ post.excerpt | strip_html | truncatewords: 50 }}

[Lire la suite]({{ post.url | relative_url }})

---

{% endfor %}