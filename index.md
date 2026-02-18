---
layout: default
title: "Articles"
---

# Articles

Technical thought pieces on AI-assisted development, architecture, and the craft of building software.

*By Steven Molen, Sr. Enterprise Architect*

---

{% assign sorted_articles = site.articles | sort: "date" | reverse %}
{% for article in sorted_articles %}

## [{{ article.title }}]({{ article.url | relative_url }})

*{{ article.subtitle }}*

{{ article.date | date: "%B %-d, %Y" }}

---

{% endfor %}
