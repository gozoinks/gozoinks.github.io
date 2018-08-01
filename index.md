---
layout: default
---

<body>

<h1>Setup Notes</h1>
<p>Notes from working with ZFS and Solaris-like systems for fun and profit, by John Burwell.</p>

<h2>Posts</h2>

<ul>{% for post in site.posts %}
<li><a href="{{ post.url }}">{{ post.title }}</a></li>{% endfor %}
</ul>

</body>
