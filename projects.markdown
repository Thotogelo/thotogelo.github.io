---
layout: page
title: Projects
permalink: /projects/
---

{% if site.data.projects and site.data.projects.size > 0 %}
<ul class="project-grid">
  {% for project in site.data.projects %}
  <li class="project-card">
    <a class="project-link" href="{{ project.url }}">
      <span class="project-title">{{ project.title }}</span>
      <span class="project-description">{{ project.description }}</span>
      {% if project.tags %}
      <span class="project-tags">{{ project.tags | join: " · " }}</span>
      {% endif %}
      <span class="project-arrow" aria-hidden="true">↗</span>
    </a>
  </li>
  {% endfor %}
</ul>
{% else %}
<p class="empty-note">Nothing here yet. The hamsters are still compiling. 🐹</p>
{% endif %}
