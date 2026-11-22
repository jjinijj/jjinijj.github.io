---
layout: page
title: Categories
permalink: /categories/
prompt: "$ ls ./projects"
---
{%- comment -%}
  프로젝트(카테고리)별 전체 글. 글이 많은 프로젝트부터, 각 프로젝트 안에서는 최신 글부터.
  #프로젝트 (slugify) 로 바로 이동할 수 있다. 홈과 글 머리의 프로젝트 칩이 이 주소로 연결된다.
{%- endcomment -%}
{%- include project-order.html -%}
{%- assign date_format = site.minima.date_format | default: "%Y.%m.%d" -%}

<nav class="chip-group category-jump" aria-label="프로젝트로 이동">
  {%- for name in project_order -%}
  <a class="tag-chip" href="#{{ name | slugify }}">{{ name | escape }}<span class="count">{{ site.categories[name].size }}</span></a>
  {%- endfor -%}
</nav>

{%- for name in project_order %}
<section class="category-section" aria-labelledby="{{ name | slugify }}">
  <div class="section-head">
    <h2 id="{{ name | slugify }}">{{ name | escape }}</h2>
    <span class="muted">{{ site.categories[name].size }}편</span>
  </div>
  {%- if site.projects[name].description %}<p class="category-desc muted">{{ site.projects[name].description | escape }}</p>{% endif %}
  <ul class="post-rows">
    {%- for post in site.categories[name] -%}
    {%- include post-title-parts.html title=post.title -%}
    <li>
      <a class="post-row" href="{{ post.url | relative_url }}">
        <span class="post-row-date mono">{{ post.date | date: date_format }}</span>
        <span class="post-row-main">
          {%- if tp_label != "" %}<span class="post-row-meta mono"><span class="post-no">{{ tp_label | escape }}</span></span>{% endif %}
          <span class="post-row-title">{{ tp_text | escape }}</span>
        </span>
      </a>
    </li>
    {%- endfor -%}
  </ul>
</section>
{%- endfor %}
