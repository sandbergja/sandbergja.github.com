---
layout: page
title: Welcome
---
I am a systems/cataloging librarian working at a community college in rural Oregon.  This blog contains updates on some of my projects, in the hopes that they will be of interest to others.

Several research interests inform my work:

* Library linked data and authority control
* Evidence-based evaluation of cataloging practices and discovery systems
* Critical cataloging
* Information needs of transgender people
* Information-seeking behavior of community college students
* The healing power of information

## List of posts
Here's what I've got:

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


