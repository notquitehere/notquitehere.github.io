---
layout: page
title: "QT Quick"
permalink: "/qtquick/"
---

<p>The 3 things todo list is based on the idea that you will be more productive if you pick just 3 things that you need to achieve each day. There are a few blog posts out there explaining the reasoning behind it such as <a href="https://www.huffpost.com/entry/the-power-of-the-three-item-to-do-list_b_9512486">The Power of the Three-Item To-Do List</a> and <a href="https://www.becomingminimalist.com/to-do/">Accomplish More with a 3-Item To Do List</a>. This tutorial series walks through making a simple version of this in QtQuick.</p>

{%- if site.qtquick.size > 0 -%}
    <ul class="post-list">
        {% for tutorial in site.qtquick %}
        <li>
            {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
            <span class="post-meta">{{ tutorial.date | date: date_format }}</span>
            <h3>
            <a class="post-link" href="{{ tutorial.url | relative_url }}">
                {{ tutorial.title | escape }}
            </a>
            </h3>
            {%- if site.show_excerpts -%}
            {{ tutorial.excerpt }}
            {%- endif -%}
        </li>
        {%- endfor -%}
    </ul>
{%- endif -%}