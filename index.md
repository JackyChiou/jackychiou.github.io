---
layout: default
title: 首頁
nav_order: 1
description: "分享在雲端世界所學的知識與實戰經驗，聚焦 Azure 平台、資安合規與雲端架構設計"
permalink: /
---

# 歡迎來到 Jacky's Cloud Blog
{: .fs-9 }

分享在雲端世界所學的知識與實戰經驗，聚焦 Azure 平台、資安合規與雲端架構設計。
{: .fs-6 .fw-300 }

[查看所有文章](/blog/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Azure 知識庫](/docs/){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## 最新文章

{% for post in site.posts limit: 5 %}
<div class="post-card">
  <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  <div class="post-meta">
    <span class="label label-blue">{{ post.date | date: "%Y-%m-%d" }}</span>
    {% for cat in post.categories %}<span class="label label-green">{{ cat }}</span> {% endfor %}
  </div>
  <p class="post-excerpt">{{ post.excerpt | strip_html | truncatewords: 35 }}</p>
</div>
---
{% endfor %}

[查看所有文章 →](/blog/){: .btn .btn-outline }
