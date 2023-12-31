---
layout: page
title: "Machine Learning"
permalink: "/machine-learning/"
---

{%- if site.machine_learning.size > 0 -%}
    <ul class="post-list">
        {% for note in site.machine_learning reversed %}
        <li>
            {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
            <span class="post-meta">{{ note.date | date: date_format }}</span>
            <h3>
            <a class="post-link" href="{{ note.url | relative_url }}">
                {{ note.title | escape }}
            </a>
            </h3>
            {%- if site.show_excerpts -%}
            {{ note.excerpt }}
            {%- endif -%}
        </li>
        {%- endfor -%}
    </ul>
{%- endif -%}