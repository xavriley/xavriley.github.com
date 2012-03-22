---
layout: page
title: Hello World!
---

# Xavier Riley

I'm a web developer working at Kyan in Guildford. Mostly Ruby, with some PHP when nothing else will do.

Here you'll find my findings, ramblings, opinions and diversions. Hope someone finds them useful, and if not useful then at least interesting.

## Recent Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

