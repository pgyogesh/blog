---
permalink: /travel/
layout: archive
author_profile: false
---


<iframe src="https://www.google.com/maps/d/u/0/embed?mid=1mPiLutLgPaihLS4zBOnrb9mupxQ" width="100%" height="480"></iframe>

{% for i in (1..tags_max) reversed %}
  {% for tag in site.tags %}
    {% if tag == "travel" %}
      <section id="{{ tag[0] | slugify | downcase }}" class="taxonomy__section">
        <h2 class="archive__subtitle">{{ tag[0] }}</h2>
        <div class="entries-{{ page.entries_layout | default: 'list' }}">
          {% for post in tag.last %}
            {% include archive-single.html type=page.entries_layout %}
          {% endfor %}
        </div>
        <a href="#page-title" class="back-to-top">{{ site.data.text[site.locale].back_to_top | default: 'Back to Top' }} &uarr;</a>
      </section>
    {% endif %}
  {% endfor %}
{% endfor %}
