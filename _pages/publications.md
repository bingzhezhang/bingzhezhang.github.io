---
layout: archive
title: "Publications"
permalink: /publications/
author_profile: true
---

Journal paper:
------
[3] Zhang, Bingzhe, et al. "Experimental and Seismic Response Study of Laminated Rubber Bearings Considering Different Friction Interfaces." Buildings 12.10 (2022): 1526.

[2] Guo, Weizuo, et al. "Research on seismic excitation direction of double-deck curved bridges: A probabilistic method based on the random forest algorithm." Structures. Vol. 39. Elsevier, 2022.

[1] Zhang, Bingzhe, et al. "Seismic response analysis and evaluation of laminated rubber bearing supported bridge based on the artificial neural network." Shock and Vibration 2021 (2021): 1-14.

Conference paper:
------
[1] Zhang, B., Wang, K. "Seismic response analysis of small-to-medium-span bridges considering aging plate rubber bearing." The 17th World Conference on Earthquake Engineering. Sendai: International Association for Earthquake Engineering. 2019. 

{% if site.author.googlescholar %}
  <div class="wordwrap">You can also find my articles on <a href="{{site.author.googlescholar}}">my Google Scholar profile</a>.</div>
{% endif %}

{% include base_path %}

{% for post in site.publications reversed %}
  {% include archive-single.html %}
{% endfor %}
