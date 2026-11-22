---
layout: page
title: Tags
permalink: /tags/
prompt: "$ ls ./tags"
---
{%- comment -%}
  태그 칩을 누르면 그 태그 글 목록과 "함께 쓰인 태그"가 바뀐다. 주소 끝 #태그 로 바로 열 수 있다.
  글 4편 이상 = 진한 블루잉크, 2~3편 = 연한 블루잉크, 1편 = 테두리만. 고른 태그는 코랄.
  스크립트가 꺼져 있으면 모든 태그 목록이 차례로 보인다.
{%- endcomment -%}
{%- capture tag_keys -%}
  {%- for tag in site.tags -%}
    {{ tag[1].size | prepend: "0000" | slice: -4, 4 }}|{{ tag[0] }}{% unless forloop.last %};;{% endunless %}
  {%- endfor -%}
{%- endcapture -%}
{%- assign tags_sorted = tag_keys | split: ";;" | sort | reverse -%}

<div class="tags-page" id="tags-page">
  <section class="card tag-cloud" aria-label="태그 목록">
    <p class="tag-total muted">태그 {{ site.tags.size }}개</p>
    <div class="chip-group">
      {%- for k in tags_sorted -%}
      {%- assign tname = k | split: "|" | last -%}
      {%- assign tn = site.tags[tname].size -%}
      <a class="tag-chip{% if tn >= 4 %} is-strong{% elsif tn >= 2 %} is-mid{% endif %}" href="#{{ tname | uri_escape }}" data-tag="{{ tname | escape }}">#{{ tname | escape }}<span class="count">{{ tn }}</span></a>
      {%- endfor -%}
    </div>
  </section>

  <div class="tags-columns">
    <div class="tags-lists">
      {%- for k in tags_sorted -%}
      {%- assign tname = k | split: "|" | last -%}
      <section class="tag-section" data-tag="{{ tname | escape }}" aria-labelledby="tag-h-{{ forloop.index }}">
        <div class="section-head">
          <h2 id="tag-h-{{ forloop.index }}" class="mono">#{{ tname | escape }}</h2>
          <span class="muted">{{ site.tags[tname].size }}편</span>
        </div>
        <ul class="post-rows">
          {%- for post in site.tags[tname] -%}
          {%- include post-title-parts.html title=post.title -%}
          <li>
            <a class="post-row is-compact" href="{{ post.url | relative_url }}" data-url="{{ post.url | relative_url }}">
              <span class="post-row-main">
                <span class="post-row-meta mono">
                  {%- if post.categories.first %}<span class="cat-chip">{{ post.categories.first | escape }}</span>{% endif -%}
                  {%- if tp_label != "" %}<span class="post-no">{{ tp_label | escape }}</span>{% endif -%}
                </span>
                <span class="post-row-title">{{ tp_text | escape }}</span>
              </span>
            </a>
          </li>
          {%- endfor -%}
        </ul>
      </section>
      {%- endfor -%}
    </div>

    <aside class="tags-side" hidden>
      <section class="card">
        <h2>함께 쓰인 태그</h2>
        <div class="chip-group tags-related"></div>
      </section>
    </aside>
  </div>
</div>

<script>
(function () {
  var root = document.getElementById('tags-page');
  if (!root) return;
  var chips = Array.prototype.slice.call(root.querySelectorAll('.tag-cloud .tag-chip'));
  var sections = Array.prototype.slice.call(root.querySelectorAll('.tag-section'));
  var side = root.querySelector('.tags-side'), related = root.querySelector('.tags-related');
  var urls = {};
  sections.forEach(function (s) {
    urls[s.dataset.tag] = Array.prototype.map.call(s.querySelectorAll('[data-url]'), function (a) { return a.dataset.url; });
  });
  root.classList.add('is-js');
  side.hidden = false;

  function select(tag) {
    if (!urls[tag]) tag = chips[0].dataset.tag;
    chips.forEach(function (c) {
      var on = c.dataset.tag === tag;
      c.classList.toggle('is-on', on);
      if (on) c.setAttribute('aria-current', 'true'); else c.removeAttribute('aria-current');
    });
    sections.forEach(function (s) { s.hidden = s.dataset.tag !== tag; });
    var mine = urls[tag], list = [];
    Object.keys(urls).forEach(function (t) {
      if (t === tag) return;
      var shared = urls[t].filter(function (u) { return mine.indexOf(u) >= 0; }).length;
      if (shared) list.push({ t: t, n: shared });
    });
    list.sort(function (a, b) { return b.n - a.n; });
    related.textContent = '';
    list.slice(0, 8).forEach(function (r) {
      var a = document.createElement('a');
      a.className = 'tag-chip is-small'; a.href = '#' + encodeURIComponent(r.t); a.textContent = '#' + r.t;
      related.appendChild(a);
    });
    side.hidden = list.length === 0;
  }
  function fromHash() { select(decodeURIComponent(location.hash.slice(1))); }
  window.addEventListener('hashchange', fromHash);
  fromHash();
})();
</script>
