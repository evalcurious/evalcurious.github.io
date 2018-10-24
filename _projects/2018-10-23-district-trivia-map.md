---
layout : research
title : District Trivia Map
excerpt_separator : <!--more-->
embed_file: district-trivia-map.html
embed_type: text/html
---
In California, my grad school friends and I got into a bit of a pub trivia kick. We even registered a team with the major trivia company in that area! Now that several of us have moved to the DC area, we still try to keep up with our weekly tradition. I designed this map just for fun as a way to help us choose where to go.

<!--more-->
The process for making this map involved:
  1. Web scraping the addresses of event locations from the trivia company's website using XML
  2. Geocoding address using MapQuest's API
  3. Generating a map using the Leaflet for R library

All of these steps were conducted in R and will be covered in tutorials on my [blog](/blog/) over the next several weeks.
