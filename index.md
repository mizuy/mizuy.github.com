---
layout: page
title: mizuy's github blog
tagline: ...
---
{% include JB/setup %}

<h2>Posts</h2>

{% for post in site.posts %}

<hr/>
<h3><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h3>
<i>{{ post.summary }}</i>
<p>
<small>{{ post.date | date_to_string }} - {{ post.category }}</small>
</p>

{% endfor %}
