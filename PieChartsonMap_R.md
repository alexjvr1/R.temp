##How to map pie-charts in R
I used this to map haplotypes on a map of Switzerland
```
require(ggplot2)
require(ggmap)
require(RgoogleMaps)
require(plotrix)
require(seqinr)
```
Read in dataset
```
setwd("~/Syncplicity/hp_1_PhD_AJvR_20130101/Analyses/Phylogeography")
path<-"~/Syncplicity/hp_1_PhD_AJvR_20130101/Analyses/Phylogeography"
cytb<-mtDNA.cytb.gps.haplo_20150506
```
set up map
```
map1.n<-paste(path,"/","map1",".png",sep="")
zoom<-8 ; lonc<-8.371714 ; latc<-46.703317 ; center<-c(latc,lonc) ; maptype<-"satellite" ; GRAYSCALE = TRUE
CH <- GetMap(center=center, zoom=zoom, maptype='satellite', destfile=map1.n, GRAYSCALE = TRUE)
CHgray = RGB2GRAY(CH)
PlotOnStaticMap(CHgray,destfile=map1.n)
```
draw pie chart
  - the addition (+0.00...1) is because R doesn't like it if the first column is empty (0). 
```
new.coords<-LatLon2XY.centered(CH,lat=cytb$lat,lon=cytb$long,zoom=zoom) #calculates new coordinates for all points
cytb$newX<-as.vector(new.coords$newX) #slightly messy way of adding new coordinates to your dataframe
cytb$newY<-as.vector(new.coords$newY)
PlotOnStaticMap(CH,destfile=map1.n)
colours<-c(col2alpha("violetred1",alpha=0.75),col2alpha("violet",alpha=0.75),col2alpha("mediumseagreen",alpha=0.75),col2alpha("mediumspringgreen",alpha=0.75)) #colours are read in order
for (i in 1:dim(cytb)[1])
{
  floating.pie(xpos=cytb$newX[i],ypos=cytb$newY[i],x=c(cytb$H1[i]+0.000000000001,cytb$H1.2[i]+0.000000000001,cytb$H2[i]+0.000000000001,cytb$H2.2[i]),
               radius=cytb$Num[i]*2,col=colours)
} #add more slices to the pie in the bracket
```


Final code used for cytb PieCharts (including brown haplotypes)
```
##Pie Charts cytb

require(ggplot2)
require(ggmap)
require(RgoogleMaps)
require(plotrix)
require(seqinr)

setwd("~/Syncplicity/hp_1_PhD_AJvR_20130101/Analyses/Phylogeography")
path<-"~/Syncplicity/hp_1_PhD_AJvR_20130101/Analyses/Phylogeography"
cytb<-mtDNA.cytb.gps.haplo_20150712

map1.n<-paste(path,"/","map1",".png",sep="")
zoom<-8 ; lonc<-8.371714 ; latc<-46.703317 ; center<-c(latc,lonc) ; maptype<-"satellite" ; GRAYSCALE = TRUE
CH <- GetMap(center=center, zoom=zoom, maptype='satellite', destfile=map1.n, GRAYSCALE = TRUE)
CHgray = RGB2GRAY(CH)
PlotOnStaticMap(CHgray,destfile=map1.n)

new.coords<-LatLon2XY.centered(CH,lat=cytb$lat,lon=cytb$long,zoom=zoom) #calculates new coordinates for all points
cytb$newX<-as.vector(new.coords$newX) #slightly messy way of adding new coordinates to your dataframe
cytb$newY<-as.vector(new.coords$newY)
PlotOnStaticMap(CH,destfile=map1.n)
colours<-c(col2alpha("darkgreen",alpha=0.75),col2alpha("yellowgreen",alpha=0.75),col2alpha("tan4",alpha=0.75),col2alpha("burlywood3",alpha=0.75),col2alpha("purple3",alpha=0.75),col2alpha("plum3",alpha=0.75)) #colours are read in order
for (i in 1:dim(cytb)[1])
{
  floating.pie(xpos=cytb$newX[i],ypos=cytb$newY[i],x=c(cytb$H1[i]+0.000000000001,cytb$H1.2[i]+0.000000000001,cytb$brown[i]+0.000000000001,cytb$brown2[i],cytb$H2[i]+0.000000000001,cytb$H2.2[i]),
               radius=cytb$Num[i]*2,col=colours)
} #add more slices to the pie in the bracket
```



