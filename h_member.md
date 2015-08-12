---
layout: page
title: 成员
comments: yes
permalink: /member/
---

<ul class="tags-box">
{% comment %}
{% for post in site.posts %}
	{% for author in post.authors %}
	&raquo; <a href="{{ site.baseurl }}	{{ author }}">	{{ author }}</a><br />

	
	{% unless author == site.authors.last %}
	{% endunless %}
	{% endfor %}
{% endfor %}
{% endcomment %}

        {% for page in site.pages %}
        {% if page.author %}
        &raquo; <a href="{{ site.baseurl }}	{{ page.author }}">	{{ page.author }}</a><br />
        	<pre>{{page.research}}</pre>
        {% endif %}
        {% endfor %}
</ul>
