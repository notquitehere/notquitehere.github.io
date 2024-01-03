---
layout: default
---
<h2>Tags</h2>
<div id="tag-cloud">
    {% assign max_posts = 0 %}

    {% for tag in site.tags %}
    {% if tag[1].size > max_posts %}
        {% assign max_posts = tag[1].size %}
    {% endif %}
    {% endfor %}

    {% assign group = max_posts | divided_by: 5.0 %}

    {% for tag in site.tags %}
    {% assign class = 5 %}
    {% for i in (1..4) %}
        {% assign boundry = group | times: i %}
        {% if tag[1].size < boundry %}
            {% assign class = i %}
            {% break %}
        {% endif %}
    {% endfor %}
    <a href="/tags/{{ tag[0] }}" class="set-{{ class }}">{{ tag[0] }}</a>
    {% endfor %}
</div>