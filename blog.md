---
layout: blog
title: "Posts"
permalink: /blog/
---

<ul class="posts">
    {% for post in site.posts %}
        <li>
            <span class="post-date">{{ post.date | date: "%b %d, %Y" }}</span>
            ::
            <a class="post-link" href="{{ post.url }}"><b>{{ post.title }}</b></a>
        </li>
    {% endfor %}
</ul>
