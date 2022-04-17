---
<<<<<<< HEAD
layout: basic 
=======
layout: default
>>>>>>> 89cac4252da40361031961c4249d5c88c87b7b1f
title: Archive
permalink: /archive
---


<ul>
	{% for post in site.posts %}
	<li><a href="{{ post.url }}" class="post-preview">{{ post.title }}</a></li>
	{% endfor %}
</ul>
