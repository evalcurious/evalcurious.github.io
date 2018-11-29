---
layout : post
title : Tutorial&#58; District Trivia Map - Part 2
subtitle: Geocoding Addresses using MapQuest's API in R
tags:
  - Mapping
  - R
  - Geocoding
  - Tutorial
---
Now that we have generated a list of addresses in [Part 1 of this series]({{ site.baseurl }}{% post_url 2018-11-30-district-trivia-map-part-1 %}), we are ready to geocode these addresses--that is, turn the addresses into the latitude and longitude coordinates we will need to map them.

The following code contains everything that is necessary to accomplish this task. There are some geocoding packages out there, for instance [Nominatim](https://www.r-bloggers.com/introducing-the-nominatim-geocoding-package/), but I had some trouble getting these packages to act as I needed them to, so I decided to geocode directly through the MapQuest API, which turned out to be less work for me.

```R
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
```
## Loading the necessary packages and address data frame
```R
# install.packages("jsonlite")
require(xml2)
require(jsonlite)

locations <- read.csv(paste0(getwd(),"//locations.csv"),stringsAsFactors = FALSE)
```
As in Part 1, we need to load the packages we'll need to simplify the task ahead. For geocoding, we will be using xml2 and jsonlite. You should already have xml2 installed if you have been following along since Part 1, but you may not have jsonlite installed. As before, I have "commented out" the code for installing jsonlite because I already have it installed. If you need to install it, remove the "#". The "require(...)" command loads the packages.

In addition, we need to load the address information we collected in Part 1. the last line of the code below reads in the csv file from our working directory. The "paste0(...)" command combines different pieces of text given to it without putting any spaces between it. The "getwd()" function provides the text version of our working directory, and the "//locations.csv" gives the name of the file that we're looking for. Finally, the "stringsAsFactors=FALSE" options tells R that we do not want it to treat text in the CSV as representing different categories. We just want the plain text. This is important because if R returns factors, instead of sending the address to the API, it might send a number representing that address as a category, and the API won't know what to do with it.

## Defining our MapQuest API key
```R
KEY <- "random number string provided by MapQuest" #your key goes here
```
To access the MapQuest API, we will need a key. An API key is a string of characters that tells the API what level of permission we have received to ask questions. Depending on the API, this could be limitations to the breadth of data you can receive, or as in this case, the API may limit the number of queries you can make in a certain period of time. Luckily the MapQuest API is pretty permissive even at the free level, providing 15,000 separate requests per month. To register and receive your API key, [follow this link](https://developer.mapquest.com/plan_purchase/steps/business_edition/business_edition_free/register).

When you get your API key, place it within the quotation marks on the line above.

## Create a geocoding function that returns a single geocoded location in JSON
```R
geocode <- function(address,key) {
  x <- gsub(" ", "+", address)
  value <- xml_text(read_html(paste0("https://www.mapquestapi.com/geocoding/v1/address?key="
                                     ,key
                                     ,"&inFormat=kvp&outFormat=json&location="
                                     ,x
                                     ,"&maxResults=1")))
  value
}
```
When we are processing data sets in this way, sometimes we will want to do the same action over and over again. R provides a nice way to do this, as do most programming languages: defining functions.

First we need to define the pieces of information we want the function to process. These appear within the "function(...)" definition in the first line--we need the function to process the address, and for that we need our API key.

Within the function (between the "{...}"), we will first use "gsub(...)" to process the address to remove the spaces and replace them with "+", because the MapQuest API expects words to be separated with a "+" rather than a space, and sometimes malfunctions if we use spaces instead.

The next few lines defines a variable called "value" that actually receives the information from the MapQuest API. We again use the xml2 packages "read_html(...)" function to pull the results of the API. The MapQuest API is accessed through a fairly typical URL, so we use "paste0(...)" to put together a URL for our query. One important setting we prepare for this query is the "maxResults=1" option. If this is not set and MapQuest cannot find an exact match to an address--which sometimes occurs with rural or new addresses--the API will return a list of addresses with the most likely address at the top of the list. We only want the top one, so we tell MapQuest to only give us one result.

The last line in the function is crucial and tells R to return the collected value once everything is done.

## Run the geocoding function on all of the Addresses
```R
locations$json <- sapply(locations$address, FUN=function(x) geocode(x,KEY))
```
Next, we define a new variable within the "locations" data frame called "json" to hold the data that the MapQuest API will return to us. Then we use the function "sapply(...)" to look at each of the addresses in the data frame one-by-one and apply our geocode function to them.

## Create a function to parse the JSON and attain latitude and longitude
```R
jsonParse <- function(x) {
  jsonList <- fromJSON(x)
  jsonList$results$locations[[1]]$latLng
}

locations$lat <- sapply(locations$json, FUN=function(x) jsonParse(x)$lat)
locations$lng <- sapply(locations$json, FUN=function(x) jsonParse(x)$lng)
```
We now have a lot more information than we need for each address in our data frame. We want to extract just the latitude and longitude for each address. Through trial and error, and use of the "str(...)" function--which reveals the structure of whatever variable to which it is applied--I figured out that after converting the JSON to a native R data format with "fromJSON(...)", the latitude and longitude were stored in "jsonList$results$locations[[1]]$latLng." This data storage location takes the form of a data frame itself. This data frame has two variables, "lat" and "lng", so once we find latLng with jsonParse(...), we will need to pull out the latitude and longitude by referring to the two variables in the data frame returned by jsonParse(x) as I have done in the next two lines. Once more, I have used sapply(...) to run a function we created for each location in our locations data set.  

## Clean up the data and save it as a CSV file
```R
locations <- locations[, !(names(locations) %in% "json")]
write.csv(locations, "locationsGeocoded.csv", row.names=FALSE)
```
Once we have our latitude and longitude coordinates, we don't need the JSON anymore. The first line in the last bit of code removes any columns in the locations data frame that have the name "json". We then use "write.csv(...)" to create a new file with our geocoded locations values.

Now that we have latitude and longitude coordinates, we are ready to put together our map. There may be some inaccuracies in the coordinates we collected, so once we have a visual of the locations, we'll be able to clean them up and finalize our map. I'll cover these steps in Part 3 next week.
