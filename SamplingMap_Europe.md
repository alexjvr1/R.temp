#Maps in R
1. Drawing basic maps
2. Drawing maps with elevation
3. Samples on map

###Good tutorials here: 
1. http://www.milanor.net/blog/?p=534
2. **MolecularEcologist Tutorial:  http://www.molecularecologist.com/2012/09/making-maps-with-r/  
3. **good for raster data: http://pakillo.github.io/R-GIS-tutorial/#intro   

###Install the following packages:

```
library(sp)  # classes for spatial data
library(raster)  # grids, rasters
library(rasterVis)  # raster visualisation
library(maptools)
library(rgeos)
# and their dependencies
```

###1. Draw a map of Europe

```
library(maps)
library(mapdata)
map(database="world", xlim=c(-10,40), ylim=c(29,75), col="gray90", fill=TRUE)
```


###2. Draw colour map of Switzerland and Sweden with elevation

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


###3. Map sample locations
To map on the elevation map: 
  1. Read in a coordinate file
  2. Plot the data
  3. Draw the elevation map over the plotted data, using alpha fxn to control transparency
  
A coordinate file would look something like this:


https://cloud.githubusercontent.com/assets/12142475/8162714/5781b6e2-137f-11e5-847d-b9721ca49c4b.png


```
#rename the file you just read into R
CH_coords<-SampledLocalities_coords_CH_20150615 
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
x <- terrain(elevation, opt = c("slope", "aspect"), unit = "degrees", xlim = 10)
plot(x)
slope <- terrain(elevation, opt = "slope")
aspect <- terrain(elevation, opt = "aspect")
hill <- hillShade(slope, aspect, 40, 270)
plot(hill, col = grey(0:100/100, alpha = 0.3), legend = FALSE, main = "Sweden")
plot(elevation, col = rainbow(25, alpha = 0.3), main = "Sweden", add = TRUE)
plot(SE_coords, pch = 20, cex = 1, col = "black", add = TRUE)


```

But the size is still wrong for the Swedish map.
And we'd like the elevational scales to be the same for the two maps. 







