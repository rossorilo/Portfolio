---
title: "Coursework"
layout: archive
permalink: /coursework/
author_profile: true
toc: true
toc_label: Coursework Contents
toc_sticky: true
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/course_kokopelli.jpg
  actions:
    - label: "Download Transcripts"
      url: "/assets/downloads/2018 Final CU and CMU Merged.pdf"
excerpt: "Miscellaneous assignments from a selection of courses."
---

{% capture written_label %}'None'{% endcapture %}

{% for collection in site.collections %}
  {% unless collection.output == false or collection.label == "posts" %}
    {% capture label %}{{ collection.label }}{% endcapture %}
    {% if label != written_label %}
      <h2 id="{{ label | slugify }}" class="archive__subtitle">{{ label }}</h2>
      {% capture written_label %}{{ label }}{% endcapture %}
    {% endif %}
  {% endunless %}
  {% for post in collection.docs %}
    {% unless collection.output == false or collection.label == "posts" %}
      {% include archive-single.html %}
    {% endunless %}
  {% endfor %}
{% endfor %}
