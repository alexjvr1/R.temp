# Maps in R
1. Drawing basic maps
2. Drawing maps with elevation
3. Samples on map

### Good tutorials here: 
1. http://www.milanor.net/blog/?p=534
2. **MolecularEcologist Tutorial:  http://www.molecularecologist.com/2012/09/making-maps-with-r/  
3. **good for raster data: http://pakillo.github.io/R-GIS-tutorial/#intro   

### Install the following packages:

```
library(sp)  # classes for spatial data
library(raster)  # grids, rasters
library(rasterVis)  # raster visualisation
library(maptools)
library(rgeos)
# and their dependencies
```

### 1. Draw a map of Europe

```
library(maps)
library(mapdata)
map(database="world", xlim=c(-10,40), ylim=c(29,75), col="gray90", fill=TRUE)
```


### 2. Draw colour map of Switzerland and Sweden with elevation

```
require(spatial.tools)
elevation<-getData("alt", country = "CH")
x <- terrain(elevation, opt = c("slope", "aspect"), unit = "degrees")
plot(x)
slope <- terrain(elevation, opt = "slope")
aspect <- terrain(elevation, opt = "aspect")
hill <- hillShade(slope, aspect, 40, 270)
plot(hill, col = grey(0:100/100), legend = FALSE, main = "Switzerland")
plot(elevation, col = rainbow(25, alpha = 0.35), add = TRUE)
```
Or change the country code to `<SE>` for Sweden, and change the country in line 7. 

![alt txt][CH_alt]

[CH_alt]: https://cloud.githubusercontent.com/assets/12142475/8160284/a5d19b84-136e-11e5-8d58-e30d28a3b42d.png

![alt txt][SE_alt]

[SE_alt]:https://cloud.githubusercontent.com/assets/12142475/8160326/efc2bb2e-136e-11e5-915e-7868f7c6c90c.png


### 3. Map sample locations
To map on the elevation map: 
  1. Read in a coordinate file
  2. Plot the data
  3. Draw the elevation map over the plotted data, using alpha fxn to control transparency
  
A coordinate file would look something like this:

![alt_txt][coordfile]

[coordfile]:https://cloud.githubusercontent.com/assets/12142475/8162714/5781b6e2-137f-11e5-847d-b9721ca49c4b.png


```
#rename the file you just read into R
CH_coords<-SampledLocalities_coords_CH_20150615
coordinates(CH_coords) <- c("Long.", "Lat.")  # set spatial coordinates
plot(CH_coords)
plot(CH_coords, pch = 20, cex = 1, col = "black")
crs.geo <- CRS("+proj=longlat +ellps=WGS84 +datum=WGS84")  # geographical, datum WGS84
proj4string(CH_coords) <- crs.geo  # define projection system of our data
summary(CH_coords)

#plot the elevation in colour
plot(elevation, col = rainbow(25, alpha = 0.3)) 

#plots the points. Symbol = pch, size = cex, add will add the plot on top of what is already there
plot(CH_coords, pch = 20, cex = 1, col = "black", add = TRUE) 
```

And for the Swedish map

```
SE_coords <-SampledLocalities_coords_SE_20150615
coordinates(SE_coords) <- c("Long.", "Lat.")  # set spatial coordinates
plot(SE_coords)
plot(SE_coords, pch = 20, cex = 1, col = "black")
crs.geo <- CRS("+proj=longlat +ellps=WGS84 +datum=WGS84")  # geographical, datum WGS84
proj4string(SE_coords) <- crs.geo  # define projection system of our data
summary(SE_coords)


require(spatial.tools)
elevation<-getData("alt", country = "SE")
x <- terrain(elevation, opt = c("slope", "aspect"), unit = "degrees")
plot(x)
slope <- terrain(elevation, opt = "slope")
aspect <- terrain(elevation, opt = "aspect")
hill <- hillShade(slope, aspect, 40, 270)
plot(hill, col = grey(0:100/100, alpha = 0.3), legend = FALSE, main = "Sweden")
plot(elevation, col = rainbow(25, alpha = 0.3), main = "Sweden", add = TRUE)
plot(SE_coords, pch = 20, cex = 1, col = "black", add = TRUE)


```

But the size is still wrong for the Swedish map.
- The size cannot be changed. The problem seems to be Sweden's dimensions.. 

And we'd like the elevational scales to be the same for the two maps. 
 - The easiest way to have similar bars is to scale Sweden's bar within the plot. 
 - Perhaps for other plots, there can be a single bar graph for the variable. But the scaling would still need to be done within the plot 

```
plot(elevation, col = (rainbow(25, start=0, end=0.375, alpha = 0.3)), legend = TRUE, main = "Sweden")
```

## Map of Sweden -> scaled and .pdf
This should be the script needed to:
1. Read in data
2. Draw the Sweden map with a scaled (elevation) graph (0.375*CH)
3. write to .pdf which will be easier to work with in Adobe Illustrator

```
SESNPs_coords <-SampledLocalities_coords_SERAD_20150616
coordinates(SESNPs_coords) <- c("Long.", "Lat.")  # set spatial coordinates
plot(SESNPs_coords)
plot(SESNPs_coords, pch = 20, cex = 1, col = "black")
crs.geo <- CRS("+proj=longlat +ellps=WGS84 +datum=WGS84")  # geographical, datum WGS84
proj4string(SESNPs_coords) <- crs.geo  # define projection system of our data
summary(SESNPs_coords)

require(spatial.tools)
elevation<-getData("alt", country = "SE")
x <- terrain(elevation, opt = c("slope", "aspect"), unit = "degrees")
plot(x)
slope <- terrain(elevation, opt = "slope")
aspect <- terrain(elevation, opt = "aspect")
hill <- hillShade(slope, aspect, 40, 270)
pdf("SERAD.pdf")
plot(hill, col = grey(0:100/100, alpha = 0.3), legend = FALSE, main = "Sweden")
plot(elevation, col = rainbow(25, start=0, end=0.375, alpha = 0.5), main = "Sweden", add = TRUE)
plot(SE_coords, pch = 20, cex = 1, col = "black", add = TRUE)
dev.off()
```



## Draw maps with different coloured samples

If the different classes of samples need to be different colours (e.g. elevation or population)

This is a good tutorial: https://sites.google.com/site/manabusakamoto/home/r-tutorials/r-tutorial-3



## Sampling map mtDNA - grayscale
```
library(sp)  # classes for spatial data
library(raster)  # grids, rasters
library(rasterVis)  # raster visualisation
library(maptools)
library(rgeos)
# and their dependencies


require(spatial.tools)
elevation<-getData("alt", country = "CH")
x <- terrain(elevation, opt = c("slope", "aspect"), unit = "degrees")
plot(x)
slope <- terrain(elevation, opt = "slope")
aspect <- terrain(elevation, opt = "aspect")
hill <- hillShade(slope, aspect, 40, 270)
plot(hill, col = grey(0:100/100), legend = FALSE, main = "Switzerland")
```


Read in the data file with an additional column specifying the colours 

R colours: http://www.stat.columbia.edu/~tzheng/files/Rcolor.pdf

```
#rename the file you just read into R
CH_coords<-CH_ddRAD_sequencedMap_Min5_20150721
coordinates(CH_coords) <- c("Long.", "Lat.")  # set spatial coordinates
plot(CH_coords)
plot(CH_coords, pch = 20, cex = 1, col = "black")
crs.geo <- CRS("+proj=longlat +ellps=WGS84 +datum=WGS84")  # geographical, datum WGS84
proj4string(CH_coords) <- crs.geo  # define projection system of our data
summary(CH_coords)

#recognise the colours column as R characters
Colour <- as.character(CH_coords$Colours)

#plots the points. Symbol = pch, size = cex, add will add the plot on top of what is already there
plot(CH_coords, pch = 20, cex = 1, col=Colour, add = TRUE) 
```

![alt txt][sequenced_20150722_alt]

[sequenced_20150722_alt]:https://cloud.githubusercontent.com/assets/12142475/8824560/3fea82c4-3077-11e5-8cea-74ec31c7c34a.png


## add text

And if you want to add labels to the sites, use the text() option

Manual can be found here: https://stat.ethz.ch/R-manual/R-devel/library/graphics/html/text.html

```
text(CH_coords, labels=CH_coords$Site, cex=0.6)
```

The full code and example map
```
##Sampling map mtDNA - grayscale

library(sp)  # classes for spatial data
library(raster)  # grids, rasters
library(rasterVis)  # raster visualisation
library(maptools)
library(rgeos)
# and their dependencies


require(spatial.tools)
elevation<-getData("alt", country = "CH")
x <- terrain(elevation, opt = c("slope", "aspect"), unit = "degrees")
plot(x)
slope <- terrain(elevation, opt = "slope")
aspect <- terrain(elevation, opt = "aspect")
hill <- hillShade(slope, aspect, 40, 270)
plot(hill, col = grey(0:100/100), legend = FALSE, main = "Switzerland")
#plot(elevation, col = rainbow(25, alpha = 0.35), add = TRUE)

##read excel sheet into R

#rename the file you just read into R
CH_coords<-CH_ddRAD_subset_Map_20150721
coordinates(CH_coords) <- c("Long.", "Lat.")  # set spatial coordinates
#plot(CH_coords)
#plot(CH_coords, pch = 20, cex = 1, col = "black")
crs.geo <- CRS("+proj=longlat +ellps=WGS84 +datum=WGS84")  # geographical, datum WGS84
proj4string(CH_coords) <- crs.geo  # define projection system of our data
summary(CH_coords)

#recognise the colours column as R characters
Colour <- as.character(CH_coords$Colours)


#plot the elevation in colour - this needs to be changed to grayscale
#plot(elevation, col = gray(1, alpha = 0.3), add=T) 

#plots the points. Symbol = pch, size = cex, add will add the plot on top of what is already there
plot(CH_coords, pch = 20, cex = 1, col=Colour, add = TRUE) 
text(CH_coords, labels=CH_coords$Site, cex=0.6)
```

![alt txt][subset_alt]

[subset_alt]:https://cloud.githubusercontent.com/assets/12142475/8824708/94418bfa-3078-11e5-8bbd-c087297f808a.png
