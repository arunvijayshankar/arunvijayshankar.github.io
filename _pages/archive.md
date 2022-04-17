---
layout: basic 
title: Archive
permalink: /archive
---


<ul>
	{% for post in site.posts %}
	<li><a href="{{ post.url }}" class="post-preview">{{ post.title }}</a></li>
	{% endfor %}
</ul>