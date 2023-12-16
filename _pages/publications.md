---
layout: page
permalink: /publications/
title: publications
description: publications by categories in reversed chronological order.
years: [2023, 2022, 2021, 2020, 2019, 2018, 2017, 2014]
nav: true
nav_order: 1
---

![A cloud tag with my research topics](/assets/img/publications.png)

_A tag cloud generated from my research papers._

<div class="publications">

{%- for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f papers -q @*[year={{y}}]* %}
{% endfor %}

</div>
