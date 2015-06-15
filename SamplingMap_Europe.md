#Maps in R
1. Drawing basic maps
2. Drawing maps with elevation
3. Samples on map

###Good tutorials here: 
http://www.milanor.net/blog/?p=534
**MolecularEcologist Tutorial:  http://www.molecularecologist.com/2012/09/making-maps-with-r/  
**good for raster data: http://pakillo.github.io/R-GIS-tutorial/#intro   

###1. Draw a map of Europe

'library(maps)
library(mapdata)
map(database="world", xlim=c(-10,40), ylim=c(29,75), col="gray90", fill=TRUE)'


###2. Draw colour map of Switzerland and Sweden with elevation
Install packages as needed (I needed to install rasterVis, rgdal, and rgeos)
require(spatial.tools)

elevation<-getData("alt", country = "CH")
x <- terrain(elevation, opt = c("slope", "aspect"), unit = "degrees")
plot(x)
slope <- terrain(elevation, opt = "slope")
aspect <- terrain(elevation, opt = "aspect")
hill <- hillShade(slope, aspect, 40, 270)
plot(hill, col = grey(0:100/100), legend = FALSE, main = "Switzerland")
plot(elevation, col = rainbow(25, alpha = 0.35), add = TRUE)

![alt txt][CH_alt]
[CH_alt]: http://127.0.0.1:30789/graphics/plot_zoom_png?width=588&height=685





