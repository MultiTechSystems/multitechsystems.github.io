---
layout: default
title: MultiTech Developer Documentation
description: Developer documentation for MultiTech IoT devices and LoRaWAN solutions
toc: false
---

<div class="home-hero">
  <h1>MultiTech Developer Documentation</h1>
  <p>Choose a documentation version:</p>
</div>

<div style="max-width: 600px; margin: 2rem auto; text-align: center;">
  {% for version in site.data.versions.versions %}
  <a href="{{ version.url }}" style="display: block; padding: 1.5rem; margin: 1rem 0; background: {% if version.current %}#0366d6{% else %}#f6f8fa{% endif %}; color: {% if version.current %}white{% else %}#24292e{% endif %}; border-radius: 8px; text-decoration: none; border: 1px solid #e1e4e8;">
    <strong>{{ version.title }}</strong>
    <br><small>{{ version.description }}</small>
    <br><small>{{ version.date }}</small>
  </a>
  {% endfor %}
</div>

<script>
  // Auto-redirect to latest version after 3 seconds
  setTimeout(function() {
    window.location.href = '/v2/';
  }, 3000);
</script>

<p style="text-align: center; color: #586069; font-size: 0.9em;">
  Redirecting to latest version in 3 seconds...
</p>
