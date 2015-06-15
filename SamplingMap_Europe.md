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






```
CH_coords<-SampledLocalities_coords_CH_20150615 #rename the file you just read into R
plot(CH_coords, pch = 20, cex = 1, col = "black") #plots the points. Symbol = pch, size = cex
plot(elevation, col = rainbow(25, alpha = 0.3), add = TRUE) #plot the elevation
```








