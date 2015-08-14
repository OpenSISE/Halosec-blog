---
layout: page
permalink: /member/Lyleaks/
author: Lyleaks
research: 網絡安全愛好者

---
<h1 class="post-title">{{page.author}}</h1>
<pre>
{{page.research}}
</pre>

<div>
{% for post in site.posts %}
	{% for author in post.authors %}
	{% if author == page.author %}
	
	&raquo; <a href="{{ post.url }}">{{post.title}}</a><br />
	{% endif %}
	{% endfor %}
{% endfor %}
</div>