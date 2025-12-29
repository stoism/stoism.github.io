---
layout: default
title: Home
permalink: /readme/ 
---

# Xin chào  
Stoism website  
Welcome!  

# DANH SÁCH BÀI VIẾT  
* [PHƯƠNG NGỮ MIỀN NAM](/phuongngumiennam.html)  
{% for page in site.pages %}
{% if page.path contains 'pages/' %}
- [{{ page.title | default: page.name }}]({{ page.url }})
{% endif %}
{% endfor %}  
* [Liên kết khác](https://raindrop.io/stodrop/stoism-link-60424721)






<p>By @stoism</p>
<p>
  <a href="/pages/about.html"><img src="https://raw.githubusercontent.com/stoism/stoism.github.io/main/assets/logo16.png" width="16" /></a> <a href="/pages/about.html">About</a>
  &mdash;
  <a href="https://github.com/stoism/stoism.github.io" target="_blank" rel="noopener noreferrer">Project</a> 
  &mdash;
  <a href="https://pages.github.com/themes" target="_blank" rel="noopener noreferrer">Theme</a>
</p>
