---
layout: page
title: Welcome
---
I am a software developer and librarian.  This blog contains very
occassional updates on some of my projects, in the hopes that they
will be of interest to others.

## List of posts
Here's what I've got:

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


