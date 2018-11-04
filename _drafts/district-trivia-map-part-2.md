---
layout : post
title : Tutorial&#58; District Trivia Map - Part 1
subtitle: Geocoding Addresses using MapQuest's API in R
tags:
  - Mapping
  - R
  - Geocoding
  - Tutorial
---

```R
# guide from https://www.r-bloggers.com/introducing-the-nominatim-geocoding-package/

# install.packages("jsonlite")

require(xml2)
require(jsonlite)

locations <- read.csv(paste0(getwd(),"//locations.csv"),stringsAsFactors = FALSE)

KEY <- "random number string provided by MapQuest" #your key goes here

geocode <- function(address,key) {
  x <- gsub(" ", "+", address)
  value <- xml_text(read_html(paste0("https://www.mapquestapi.com/geocoding/v1/address?key="
                                     ,key
                                     ,"&inFormat=kvp&outFormat=json&location="
                                     ,x
                                     ,"&maxResults=1")))
  value
}

locations$json <- sapply(locations$address, FUN=function(x) geocode(x,KEY))

jsonParse <- function(x) {
  jsonList <- fromJSON(x)
  jsonList$results$locations[[1]]$latLng
}

locations$lat <- sapply(locations$json, FUN=function(x) jsonParse(x)$lat)
locations$lng <- sapply(locations$json, FUN=function(x) jsonParse(x)$lng)

locations <- locations[, !(names(locations) %in% "json")]

write.csv(locations, "locationsGeocoded.csv", row.names=FALSE)

# foo.json <- geocode("7301 Calhoun PI #600, Derwood, MD 20855",KEY)
```
