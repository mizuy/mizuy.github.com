---
layout: page
title: Hello World!
tagline: Supporting tagline
---
{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


<h2>Posts</h2>

{% for post in site.posts %}

<hr/>
<h3><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h3>
<i>{{ post.summary }}</i>
<p>
<small>{{ post.date | date_to_string }} - {{ post.category }}</small>
</p>

{% endfor %}
