---
layout: page
title: My photosets 📸
permalink: /gallery/
---

<h1>{{ page.title }}</h1>

<p>I've been lucky enough to travel to some beautiful places, and in order to document that, I started taking amateur photographs which you can find below 👇</p>

{% assign count = 0 %}
{% assign align = "left" %}
{% for gallery in site.data.galleries.overview %}
{% if count == 0 %}<div class="row">{% endif %}
	<div class="half-width gallery-preview {{ align }}">
		<h1>{{ gallery.title }}</h1>
		<a href="/gallery/{{ gallery.directory }}">
			<img alt="{{ gallery.title }}" src="/assets/gallery/{% if gallery.picture_path %}{{ gallery.picture_path }}{% else %}{{ gallery.directory }}{% endif %}/{{ gallery.preview.thumbnail }}" />
		</a>
	</div>
{% if count == 1 %}</div>{% endif %}
{% assign count = count | plus: 1 %}
{% assign align = "right" %}
{% if count >= 2 %}
{% assign align = "left" %}
{% assign count = 0 %}
{% endif %}
{% endfor %}