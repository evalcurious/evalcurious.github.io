---
layout : post
title : Tutorial&#58; District Trivia Map - Part 3
subtitle: Mapping Geocoded Addresses with Leaflet for R
tags:
  - Mapping
  - R
  - Leaflet
  - Tutorial
---
[Part 1 of this series]({{ site.baseurl }}) [Part 2 of this series]({{ site.baseurl }})

```R
# install.packages("leaflet")
require(leaflet)

trivia <- read.csv(paste0(getwd(),"/locationsGeocoded.csv"))

trivia$address <- gsub(" ?,(?=( +[A-z]+)+ ?,(.+))","<br>",trivia$address,perl=TRUE)

map <- leaflet(trivia) %>%
  addProviderTiles(provider="Stamen.Watercolor") %>%
  addProviderTiles(provider="CartoDB.PositronOnlyLabels") %>%
  addMarkers(lat=~lat,lng=~lng
                   ,group=~day
                   ,popup=~paste0("<h4>",name,"</h4>",
                                  "<p><em>",time,"</em></p>",
                                  address
                                  )) %>%
  addLayersControl(overlayGroups = c("Monday","Tuesday","Wednesday","Thursday","Sunday")
                   ,options=layersControlOptions(collapsed=FALSE)) %>%
  setView(lng=-77.0369,lat=38.9072,zoom=9)
```

## Load required packages and geocoded address data
```R
# install.packages("leaflet")
require(leaflet)

trivia <- read.csv(paste0(getwd(),"/locationsGeocoded.csv"))
```

## Format addresses for readability
```R
trivia$address <- gsub(" ?,(?=( +[A-z]+)+ ?,(.+))","<br>",trivia$address,perl=TRUE)
```
Tested regex at https://regexr.com/

## Generate map with Leaflet
```R
map <- leaflet(trivia) %>%
  addProviderTiles(provider="Stamen.Watercolor") %>%
  addProviderTiles(provider="CartoDB.PositronOnlyLabels") %>%
  addMarkers(lat=~lat,lng=~lng
                   ,group=~day
                   ,popup=~paste0("<h4>",name,"</h4>",
                                  "<p><em>",time,"</em></p>",
                                  address
                                  )) %>%
  addLayersControl(overlayGroups = c("Monday","Tuesday","Wednesday","Thursday","Sunday")
                   ,options=layersControlOptions(collapsed=FALSE)) %>%
  setView(lng=-77.0369,lat=38.9072,zoom=9)
```

## Address data cleaning

## Exporting the map as HTML
