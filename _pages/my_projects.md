---
layout: page
title: projects
permalink: /projects/
description: A growing collection of my personal projects.
nav: true
nav_order: 1
display_categories:
  - title: GINS
    single_col: false
  - title: SLAM
    single_col: false
  - title: Localization
    single_col: false
  - title: Mapping & Planning
    single_col: false
  - title: ROS
    single_col: true
  - title: Multi-robot System
    single_col: true
  - title: Computer Vision
    single_col: false
  - title: Machine Learning
    single_col: false
horizontal: false
---

<!-- pages/projects.md -->
<div class="projects">
{% if site.enable_project_categories and page.display_categories %}
  <!-- Display categorized projects -->
  {% for category in page.display_categories %}
    <a id="{{ category.title }}" href=".#{{ category.title }}">
    <h2 class="category">{{ category.title }}</h2>
    </a>
    {% assign categorized_projects = site.projects | where: "category", category.title %}
    {% assign sorted_projects = categorized_projects | sort: "importance" %}
    <!-- Generate cards for each project -->
    {% if category.single_col%}
      <div class="container">
        <div class="row row-cols-1 row-cols-md-1">
          {% for project in sorted_projects %}
            {% include projects.liquid %}
          {% endfor %}
        </div>
      </div>
    {% else %}
      <div class="row row-cols-1 row-cols-md-2">
        {% for project in sorted_projects %}
          {% include projects.liquid %}
        {% endfor %}
      </div>
    {% endif %}
    {% endfor %}
{% else %}
  <!-- Display projects without categories -->
  {% assign sorted_projects = site.projects | sort: "importance" %}
  <!-- Generate cards for each project -->
  {% if page.horizontal %}
    <div class="container">
      <div class="row row-cols-1 row-cols-md-2">
      {% for project in sorted_projects %}
        {% include projects_horizontal.liquid %}
      {% endfor %}
      </div>
    </div>
  {% else %}
    <div class="row row-cols-1 row-cols-md-3">
      {% for project in sorted_projects %}
        {% include projects.liquid %}
      {% endfor %}
    </div>
  {% endif %}
{% endif %}
</div>
