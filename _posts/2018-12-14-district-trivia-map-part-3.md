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
In [Part 1 of this series]({{ site.baseurl }}{% post_url 2018-11-30-district-trivia-map-part-1 %}), we used some web scraping techniques to collect a list of addresses. In [Part 2]({{ site.baseurl }}{% post_url 2018-12-07-district-trivia-map-part-2 %}), we geocoded these addresses with MapQuest's API to find coordinates in latitude and longitude format. Now that we have a set of coordinates, we are ready to build an interactive web map using Leaflet for R. The following code is all we need to build a nice looking map.

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
As is always the case, we first need to install and load the packages necessary to run our code. The package we'll be using is leaflet for R, which is a map-building javascript library adapted by the RStudio folks to be applied within an R environment. The project documentation for [leaflet for R can be found here](https://rstudio.github.io/leaflet/).

After loading the leaflet package, we will read our geocoded locations file into a variable called "trivia", using the getwd function to ensure that the file is being selected from our working directory.

## Format addresses for readability
```R
trivia$address <- gsub(" ?,(?=( +[A-z]+)+ ?,(.+))","<br>",trivia$address,perl=TRUE)
```
I decided that when I generate the map, I want to have pop-up labels show up for each trivia location on the map that show the address, date, and time of the event there. The only problem is that the addresses aren't in a great format for readability, most of them are currently written out as:
```
1234 Main St, Local City, ZZ 12345
```
To format these more in the way people are accustomed to seeing addresses, and to reduce the potential width of the pop-up labels, I want to find a way to quickly edit all of these so that the are formatted as:
```
1234 Main St
Local City, ZZ 12345
```

Essentially, I want to introduce a line break after the first comma. Since the final output of the map will be in HTML, we can use the "\<br>" tag to define this line break. The gsub(...) command in r is a good way of replacing one character string with another character string, but using the following command:
```R
gsub(",","<br>",trivia$address)
```
is going to result in a line break at every comma.
```
1234 Main St
Local City
ZZ 12345
```
In the US, this is a rather non-standard way of writing out an address.

To remedy this situation, I use a regex string instead of a simple "," as our search text for our search-and-replace function. I usually test my regex online before I run it, for example at [this website](https://regexr.com/441uq). I have shared the regex string I used with a handful of sample addresses so that you can play around and see how it works. If you are familiar with regex, you will notice that I included some operators to account for extra spaces, this is because I discovered by trial-and-error that many of the addresses on the District Trivia website have editing mistakes in them and I had to adjust the regex so that it wouldn't break on these addresses.

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
Now we are ready to generate a map. To make sense of this code, you should know that Leaflet uses the convenient "%>%" pipe operator from the [magrittr package](https://cran.r-project.org/web/packages/magrittr/vignettes/magrittr.html). Without this, we would have to re-define the "map" variable on each line. This operator helps us develop some cleaner looking code.

First, we want to create a variable to hold our map, and load our address data into the leaflet map-space.
```R
map <- leaflet(trivia) %>%
```
Next, we want to lay down some map tiles, which provide the background. Most of the time, you only need to load one set of tiles, but "Stamen.Watercolor" does not come with location and street labels, so I am also loading a set of titles that only includes these labels. You can find a full list of provider tiles on the [leaflet for R](https://rstudio.github.io/leaflet/) website.
```R
addProviderTiles(provider="Stamen.Watercolor") %>%
addProviderTiles(provider="CartoDB.PositronOnlyLabels") %>%
```
Then it is time to add markers for each of our locations. Since we already loaded the trivia data variable on the first line of the map definition code, we don't need to call it again. We can instead reference data columns within that variable with "~". I define the latitude and longitude variables, then I tell leaflet that I want to group the markers by the day of the week the event takes place. Finally, I use a bit of HTML code within a "paste0(...)" function to define what I want the pop-up to look like.
```R
addMarkers(lat=~lat,lng=~lng
                 ,group=~day
                 ,popup=~paste0("<h4>",name,"</h4>",
                                "<p><em>",time,"</em></p>",
                                address
                                )) %>%
```
Because I told leaflet I wanted to group the markers, I can add something called a layer control. This allows me to treat each day-of-the-week grouping as a separate layer of markers, and allows me to filter out all but one day's worth of events at a time. This control appears a menu of check-boxes in the upper right hand corner of the page.
```R
addLayersControl(overlayGroups = c("Monday","Tuesday","Wednesday","Thursday","Sunday")
                 ,options=layersControlOptions(collapsed=FALSE)) %>%
```
Finally, in completing this map, I discovered that there are a couple of locations that I am personally not particularly interested in way down south in Virginia. Since leaflet sets the view window automatically, these locations cause the map to be quite zoomed out, which makes it hard to distinguish between the locations I am actually interested in. So I set the view area to be centered on a more convenient area and zoomed the map in until I liked how it looked.
```R
setView(lng=-77.0369,lat=38.9072,zoom=9)
```

## Note: Address data cleaning
Even if you do everything correctly up to this point, you may still need to look at the final output to check and make sure everything *looks* okay. I discovered after constructing my map, that there was one geocoded coordinate that was way out in the Midwest somewhere. I double checked on the District Trivia website that they only serve DC, Maryland, and Virginia and checked the address in question to see if it was correct. The address that caused the problem was:
```
7301 Calhoun PI #600, Derwood, MD 20855
```
In some fonts it is hard to tell, but instead of PL as in "place", someone had mistyped the address as PI For some reason, when accessing the MapQuest API through R, this typo was resulting in an address way out west. If you replicate this process for another website, you may come across similar problems. I addressed it manually, but if you are dealing with a large number of websites, you may be able to address it through an address correction API like [this one by UPS](https://www.ups.com/us/en/services/technology-integration/us-address-validation.page).

## The Final Map
Click on the map below to see an interactive version on my [projects page]({% link projects.html %}).
[![Static image of the final map]({% link /assets/img/districtTriviaMapStatic.png %})]({% link _projects/2018-10-23-district-trivia-map.md %})
