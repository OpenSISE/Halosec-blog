---
layout: page
permalink: /member/p74n3_1C4tr0/
author: p74n3_1C4tr0
research: Reverse Engineering ..

---
<h1 class="post-title">{{page.author}}</h1>
![LCatro](http://ww3.sinaimg.cn/large/6a87387agw1ev2ox9qccsj20cs0csdic.jpg)
<pre>
{{page.research}}
</pre>


<br />
<a href="https://github.com/LCatro" title="GithubID: LCatro"><i class="fa fa-github-square fa-4x"></i></a><br />

<div>
{% for post in site.posts %}
	{% for author in post.authors %}
	{% if author == page.author %}
	
	&raquo; <a href="{{ post.url }}">{{post.title}}</a><br />
	{% endif %}
	{% endfor %}
{% endfor %}
</div>