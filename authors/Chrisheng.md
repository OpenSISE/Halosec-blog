---
layout: page
permalink: /member/Chr1sh3ng/
author: Chr1sh3ng
research: Web安全、虚拟化狂热者

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