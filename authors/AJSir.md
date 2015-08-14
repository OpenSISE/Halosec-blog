---
layout: page
permalink: /member/AJSir/
author: AJSir
research: 深藏不露的全栈工程师

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