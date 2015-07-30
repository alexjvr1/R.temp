#Genotype-Environment association analyses

##WorldClim data

One fo the first things to do is to obtain WorldClim data for all the sample locations. This can be done in R. 

This can be obtained either by: 

1. GetData: http://pakillo.github.io/R-GIS-tutorial/

2. rWBclimate: https://cran.r-project.org/web/packages/rWBclimate/rWBclimate.pdf


Blog about getting WorldClim data into R: 

https://ecologicaconciencia.wordpress.com/2013/11/29/obtaining-macroclimate-data-with-r-to-model-species-distributions/


I will download 30 arc seconds data.

- global data will be downloaded for all except 0.5 (30 arc sec). For this, lon & lat need to be specified. Co-ordinates within the tile of interest. 
- Tiles can be seen here: http://www.worldclim.org/tiles.php

```
library(rgdal)
library(raster)
tile116 <- getData('worldclim', var="bio", res=0.5, lon=3.8, lat=48.3)
```

Now import the raster file into R and plot it: 
```
tminCM <- raster("/Users/alexjvr/phd_20150212/Analysis/ddRAD/wc0.5/bio6_16.bil") 
plot(tminCM)
```

![alt_txt][bio6]
[bio6]:https://cloud.githubusercontent.com/assets/12142475/8985981/2f9707d0-36da-11e5-8add-22e56497821f.png





