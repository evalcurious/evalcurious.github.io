---
layout : research
title : District Trivia Map
excerpt_separator : <!--more-->
embed_file: district-trivia-map.html
embed_type: text/html
date_1: 2018-11-30 00:00:00
date_2: 2018-12-07 00:00:00
date_3: 2018-12-14 00:00:00
---
In California, my grad school friends and I got into a bit of a pub trivia kick. We even registered a team with the major trivia company in that area! Now that several of us have moved to the DC area, we still try to keep up with our weekly tradition. I designed this map just for fun as a way to help us choose where to go.

<!--more-->
The process for making this map involved:
  1. [Web scraping]({{ site.baseurl }}{% post_url 2018-11-30-district-trivia-map-part-1 %}) the addresses of event locations from the trivia company's website using XML
  2. {% if site.time >= page.date_2 %}[Geocoding]({{ site.baseurl }}{% post_url 2018-12-07-district-trivia-map-part-2 %}){% else %}Geocoding{% endif %} addresses using MapQuest's API
  3. {% if site.time >= page.date_3 %}[Generating]({{ site.baseurl }}{% post_url 2018-12-14-district-trivia-map-part-3 %}){% else %}Generating{% endif %} a map using the Leaflet for R library

All of these steps were conducted in R and will be covered in tutorials on my [blog](/blog/) over the next several weeks.
