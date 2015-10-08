#Scripts that have not been fully optimised

##Tab 1

```
#Check that R is at least v.2.12.0
R.version.string
library(adegenet)
packageDescription("adegenet", fields = "Version")
require(pegas)
library(ade4)
library(ape)
library(genetics)
library(hierfstat)


help.search("Hardy-Weinberg")
help.search("pca")
'?'(spca)

setwd("~/phd_20150212/Analysis/ddRAD")

getClassDef("SNPbin")
getClassDef("genlight")

#read in genepop file
subset <- read.genepop("subset.final.maf25.pop.gen")
nancycats <- read.genepop("nancycats.gen")
data(nancycats)
subset

##convert to hierfstat format
dat <- genind2hierfstat(subset)

##hierfstat with sam's sunflower data

library (hierfstat)

#change the working directory as needed
input <- read.table ("ann_pet_fst_map_wc",T)
input 

#extract the population names:
popnames <- colnames (input[,4:66])
popnames

#convert them to pop_id
pop_id <- array (0,length (popnames))
pop_id[grep("ann",popnames)] <- 1
pop_id[grep("pet",popnames)] <- 2
pop_id

#reformat the data:
transpose <- t(rbind (pop_id, as.matrix (input[,4:66])))

#sample only the first 500 loci (it takes too long to calculate FST otherwise), and reformat them for hierfstat format:
bb <- cbind (as.character (pop_id), as.matrix (transpose[,2:501]))
bb[bb== "XX"] <- "NA"
bb[bb=="AA"] <- "11"
bb[bb=="AC" | bb=="CA"] <- "12"
bb[bb=="AG" | bb=="GA"] <- "13"
bb[bb=="AT" | bb=="TA"] <- "14"
bb[bb=="CC"] <- "22"
bb[bb=="CG"| bb=="GC"] <- "23"
bb[bb=="CT"| bb=="TC"] <- "24"
bb[bb=="GG"] <- "33"
bb[bb=="GT"| bb=="TG"] <- "34"
bb[bb=="TT"] <- "44"


#this is a quick and dirty way to make sure the formatting is right. I like how read.table takes care of the annoying problem of making things factors (if you know a better way, let me know!). Note: this is slow when there are many columns, R not designed for that format of data.
write.table (bb, "trash1.txt", row.names = F, col.names = F, quote = F)
data <- read.table ("trash1.txt")

#calculate basic statistics:
stats <- basic.stats(subset)
stats

#calculate Weir and Cockerham (1984) FST:
wc_stats <- wc (data)
wc_stats$sigma.loc

a <- mean(as.numeric(wc_stats$lsiga))


b <- sum(wc_stats$lsigb)

c <- sum(wc_stats$lsigw)
c

abc <- mean(a+b+c)

abc

#get the per locus stats:
stats$perloc

#get the per locus stats:
wc_stats$per.loc$FST

#get the variance components:
wc_stats$sigma.loc
###check the headers of various components of the objects when you type "wc_stats" or "stats", so you can see how to extract different elements of the data


##hierfstat to calc fst
require(hierfstat)

stats <- basic.stats(dat)

stats

#calculate Weir and Cockerham (1984) FST:
wc_stats <- wc (data)

#get the per locus stats:
stats$perloc

#get the per locus stats:
wc_stats$per.loc$FST

#get the variance components:
wc_stats$sigma.loc






adegenetTutorial("basics")

##read .vcf file in

glPlot(subset, posi="topleft")
glPlot(nancycats, posi="topleft")
subset

dat <- data.frame(locus1 = c("A/T", "T/T"), locus2 = c("G/G","C/G"))
dat
x <- df2genind(dat, sep = "/")

x

subset@ind.names

##HWE by pop and plot

subset[1:10, ]

subset <- HWE.test.genind(subset, res.type = "matrix")

barplot(apply(subset < 0.001, 1, mean, na.rm = TRUE), las = 3, names.arg= c("arce(Low)", "bide(High)", "csan(High)", "gola(Mid)", "kand(Low)", "lens(Mid)", "meie(High)", "orge(Low)", "shwe(Mid)", "star(High)", "tsee(Mid)", "wise(Low)"),  ylab = "Proportion of departures from HWE", main = "By population")


###fstats

library(hierfstat)
library(numpy)

fstat

lsigma <- as.numeric(wc_stats$lsiga)
summary(lsigma)
barplot(lsigma)

lsigma <- as.numeric(wc_stats$lsiga)
lsigma


a <- mean(lsigma)
a

gstat.randtest






##Draw a tree based on genetic distances

data(subset)
X <- na.replace(subset)$tab




glPlot("nancycats")


showClass("subset")
```




##Tab 2
```
###He using adegenet

#http://www.inside-r.org/packages/cran/adegenet/docs/Hs

library(adegenet)

#import genepop file
subset <- read.genepop("subset.final.maf25.pop.gen")
subset <- 

#nuc diversity
library(pegas)
nuc.div(subset, variance = F, pairwise.deletion =F)

subsetHs <- Hs(subset, truenames=T)
subsetHs

```


##Tab 3
```
##RDA - Victoria Sork

library(vegan)
library(permute)

###Import WorldClim data

library(rgdal)
library(raster)
tile116 <- getData('worldclim', var="bio", res=0.5, lon=3.8, lat=48.3)

#read bio6 data frame into R & plot
tminCM <- raster("bio06_16.bil") 
plot(tminCM)

##If data for co-ordinate points are needed: 
#import data
subset.coords <- subseet_coords

xy <- cbind(subseet_coords$lon, subseet_coords$lat)
test <- extract(tminCM, xy)
test


sp <- SpatialPoints(xy)

summary(sp)
sp 

sites_cli <- as.data.frame(cbind(subset.coords,"bio06" = extract(tminCM, sp, method='bilinear')))

library(raster)
sites_cli <- extract(tminCM, sp, method="bilinear")
        
  
summary(sites_cli)
barplot(sites_cli)
sites_cli

library(BioCalc)
?BioCalc

```


##Tab 4
```
#Check that R is at least v.2.12.0
R.version.string
library(adegenet)
packageDescription("adegenet", fields = "Version")
require(pegas)
library(ade4)
library(ape)
library(genetics)
library(hierfstat)

setwd("~/phd_20150212/Analysis/ddRAD")

getClassDef("SNPbin")
getClassDef("genlight")

#read in genepop file
subset <- read.genepop("subset.final.maf25.pop.gen")
subset

##HWE
temp <- HWE.test.genind(subset, res.type = "matrix")
barplot(apply(temp < 0.001, 1, mean, na.rm = TRUE), las = 3, ylab = "Proportion of departures from HWE", main = "By population")

barplot(apply(temp < 0.001, 2, mean, na.rm = TRUE), las = 3, ylab = "Proportion of departures from HWE", main = "By marker")

gst <- gstat.randtest(subset)
barplot(gst)


#PCA

X <- scaleGen(subset, scale = F, miss="mean")
pcaX <- dudi.pca(X, cen = FALSE, scale = FALSE, scannf = FALSE, nf = 3)

summary(X)
barplot(pcaX$eig, main = "Eigenvalues")
pcaX

s.label(pcaX$li, clab = 0.75)
add.scatter.eig(pcaX$eig, 3, 1, 2)
myCol <- rainbow(length(levels(pop(subset))))

par(bg = "grey")
s.class(pcaX$li, pop(subset), col = myCol)
add.scatter.eig(pcaX$eig, 3, 1, 2)
```


##Tab 6
```

het <- pyrad.out.stats
het <- read.csv("pyrad.out.stats.csv", header = TRUE, sep = ",", quote = "\"",dec = ".", fill = TRUE, comment.char = "")

require(ggplot2)

qplot(het$Canton, het$prop.maf10, fill=factor((Height), levels=c("Low","Mid","High")), data=het, geom="boxplot", position="dodge") + xlab("Transect") + ylab("Inbreeding coefficient (F)")+ ggtitle("Inbreeding coefficient for each elevational transect in Switzerland") + scale_fill_manual(values=c("lightgoldenrod1","olivedrab2","royalblue"))+theme_bw()+theme(legend.title=element_blank())

```

##Tab 7
```
# 	 This file is used to plot figures for the software Bayescan in R.

#    This program, BayeScan, aims at detecting genetics markers under selection,
#	 based on allele frequency differences between population. 
#    Copyright (C) 2010  Matthieu Foll
#
#    This program is free software: you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation, either version 3 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.

#    You should have received a copy of the GNU General Public License
#    along with this program.  If not, see <http://www.gnu.org/licenses/>.

# Arguments:
# - file is the name of your file ex: "output_fst.txt"
# - the q-value threshold corresponding to the target False Discovery Rate (FDR)
# - size is the size of the points and text labels for outliers
# - pos is the distance between the points and the labels 
# - highlight is a optional list of marker indices to display in red.
# - name_highlighted alows to write the indices of highlighted markers instead of using a point like the other markers
# - add_text adds the indices of the outlier markers

# Output:
# This function returns different paremeters in a list
# - outliers: the list of outliers
# - nb_outliers: the number of outliers

# Typical usage: 
# - load this file into R (file/source R code)
# - in R, go to the directory where "output_fst.txt" is (file/change current dir)
# - at the R prompt, type 
# > plot_bayescan("output_fst.txt",0,FDR=0.05)
# if you save the output in a variable, you can recall the different results:
# results<-plot_bayescan("output_fst.txt",0,FDR=0.05)
# results$outliers
# results$nb_outliers

#
# plotting posterior distribution is very easy in R with the output of BayeScan:
# first load the output file *.sel produced by BayeScan
# > mydata=read.table("bi.sel",colClasses="numeric")
# choose the parameter you want to plot by setting for example:
# parameter="Fst1"
# then this line will make the plot for:
# > plot(density(mydata[[parameter]]),xlab=parameter,main=paste(parameter,"posterior distribution"))
# you can plot population specific Fst coefficient by setting
# parameter="Fst1"
# if you have non-codominant data you can plot posterior for Fis coefficients in each population:
# parameter="Fis1"
# if you test for selection, you can plot the posterior for alpha coefficient for selection:
# parameter="alpha1"
# you also have access to the likelihood with:
# parameter="logL"
# if you have the package "boa" installed, you can very easily obtain Highest Probability 
# Density Interval (HPDI) for your parameter of interest (example for the 95% interval):
# > boa.hpd(mydata[[parameter]],0.05)


plot_bayescan<-function(res,FDR=0.05,size=1,pos=0.35,highlight=NULL,name_highlighted=F,add_text=T)
{
if (is.character(res))
  res=read.table(res)

colfstat=5
colq=colfstat-2

highlight_rows=which(is.element(as.numeric(row.names(res)),highlight))
non_highlight_rows=setdiff(1:nrow(res),highlight_rows)

outliers=as.integer(row.names(res[res[,colq]<=FDR,]))

ok_outliers=TRUE
if (sum(res[,colq]<=FDR)==0)
	ok_outliers=FALSE;

res[res[,colq]<=0.0001,colq]=0.0001

# plot
plot(log10(res[,colq]),res[,colfstat],xlim=rev(range(log10(res[,colq]))),xlab="log10(q value)",ylab=names(res[colfstat]),type="n")
points(log10(res[non_highlight_rows,colq]),res[non_highlight_rows,colfstat],pch=19,cex=size)

if (name_highlighted) {
 	if (length(highlight_rows)>0) {
 		text(log10(res[highlight_rows,colq]),res[highlight_rows,colfstat],row.names(res[highlight_rows,]),col="red",cex=size*1.2,font=2)
 	}
}
else {
	points(log10(res[highlight_rows,colq]),res[highlight_rows,colfstat],col="red",pch=19,cex=size)
	# add names of loci over p and vertical line
	if (ok_outliers & add_text) {
		text(log10(res[res[,colq]<=FDR,][,colq])+pos*(round(runif(nrow(res[res[,colq]<=FDR,]),1,2))*2-3),res[res[,colq]<=FDR,][,colfstat],row.names(res[res[,colq]<=FDR,]),cex=size)
	}
}
lines(c(log10(FDR),log10(FDR)),c(-1,1),lwd=2)

return(list("outliers"=outliers,"nb_outliers"=length(outliers)))
}

```

##Tab 8
```
#Sampling map SNPs - grayscale

library(sp)  # classes for spatial data
library(raster)  # grids, rasters
library(rasterVis)  # raster visualisation
library(maptools)
library(rgeos)
# and their dependencies


SESNPs_coords <-SE_coords_HiSeq
coordinates(SESNPs_coords) <- c("Long.", "Lat.")  # set spatial coordinates
plot(SESNPs_coords)
plot(SESNPs_coords, pch = 20, cex = 1, col = "black")
crs.geo <- CRS("+proj=longlat +ellps=WGS84 +datum=WGS84")  # geographical, datum WGS84
proj4string(SESNPs_coords) <- crs.geo  # define projection system of our data
summary(SESNPs_coords)

require(spatial.tools)
elevation<-getData("alt", country="DNK")
x <- terrain(elevation, opt = c("slope", "aspect"), unit = "degrees")
plot(x)
#getData("ISO3")
slope <- terrain(elevation, opt = "slope")
aspect <- terrain(elevation, opt = "aspect")
hill <- hillShade(slope, aspect, 40, 270)
#pdf("SERAD.pdf")
plot(hill, col = grey(0:100/100, alpha = 0.3), legend = FALSE, main = "Denmark")
#plot(elevation, col = rainbow(25, start=0, end=0.375, alpha = 0.5), main = "Sweden", add = TRUE)
plot(SESNPs_coords, pch = 20, cex = 1, col = "black", add = TRUE)
#dev.off()

```

##Tab 9

This wasn't run (didn't work)

```
## Not run: 
library('raster')
border = raster::getData('GADM',country='CHE',level=0)

myextent = extent(5 ,12,45,49)

theres=10
rain = raster::getData('worldclim', var='prec', res=theres,mask=FALSE)#,lon=20,lat=47)
rain = raster::crop(rain,myextent)
plot(rain[['prec3']])
plot(border,add=TRUE)

alt = raster::getData('worldclim', var='alt',res=theres)
alt = raster::crop(alt,extent(rain))
plot(alt)
plot(border,add=TRUE)


bio = raster::getData('worldclim', var='bio',res=theres)
bio = raster::crop(bio,extent(rain))
plot(alt)
plot(border,add=TRUE)

swissRainR = rain
#	rain[[c('prec2','prec7')]]
swissRainR = addLayer(swissRainR,alt)	

library('geostatsp')
swissNN = NNmat(swissRainR)

save(swissRainR, swissNN,file=
       "/home/patrick/workspace/diseasemapping/pkg/gmrf/data/swissRainR.RData",
     compress='xz'	)	


## End(Not run)
data('swissRainR')
plot(swissRainR[['prec7']])

plot(swissRainR[['alt']])

swissNN[1:4,1:5]

[Package geostatsp version 1.3.4 Index]

```


##Not sure what this code was for
```
library(GEOmap)
data(ETOPO5)
data(fujitopo)
data(japmap)
PLOC=list(LON=c(137.008, 141.000),LAT=c(34.000, 36.992),
          x=c(137.008, 141.000), y=c(34.000, 36.992) )
JAPANtopo = subsetTOPO(ETOPO5, PLOC)
d1 = dim(JAPANtopo$z)
```

##Great circle distance between samples
```
##Great circle distance between samples

##Geographic distance between all sequenced R.temp CH samples
##using package Rdist

library(fields)
#Import .csv with coordinates
CH_RAD_coords_20151001

#rdist.earth (in fields package) wants only long & lat
CH_RAD_lon.lat <- cbind(CH_RAD_coords_20151001$Long., CH_RAD_coords_20151001$Lat.)
CH_RAD_lon.lat

#calculate great circle distances
distance.matrix.CHRAD <- rdist.earth(CH_RAD_lon.lat)
summary(distance.matrix.CHRAD)
dim(distance.matrix.CHRAD)

#and use only the lower half of the matrix
upper.tri(distance.matrix.CHRAD)
distance.matrix.CHRAD[lower.tri(distance.matrix.CHRAD)]<-NA
distance.matrix.CHRAD

#change from matrix to dataframe
bli <- as.data.frame(distance.matrix.CHRAD)
head(bli)
colnames(bli) <- CH_RAD_coords_20151001$Site
rownames(bli) <- CH_RAD_coords_20151001$Site

bli[lower.tri(bli,diag=TRUE)]=NA  #Prepare to drop duplicates and meaningless information
bli=as.data.frame(as.table(as.matrix(bli)))  #Turn into a 3-column table
bli
bli=na.omit(bli)  #Get rid of the junk we flagged above
bli
colnames(bli)<-c("site1", "site2", "dist(km)")
head(bli)

bli2 <- bli[sort(site2),]

##write to csv
write.csv(bli, file="distance.CHRAD.csv",row.names=F)




#use melt to reorder the data ***This doesn't work for the comparison matrix, but useful for data with variable columns
install.packages("reshape2")
install.packages("reshape")
library(reshape)
library(reshape2)
head(bli)
summary(bli)
bli2 <-na.omit(bli) #remove NAs
head(bli2)
summary(bli2)

z<-as.data.frame(as.table(bli))

test <- melt(bli2)
head(test)
tail(test)



##write to csv
write.csv(distance.matrix.CHRAD, file="distance.matrix.CHRAD.csv",row.names=F)
summary(distance.matrix.CHRAD)
dim(distance.matrix.CHRAD)


##I think this part is to get one part of the distance table

system.time(correlations<-cor(mydata,use="pairwise.complete.obs"))#get correlation matrix
upperTriangle<-upper.tri(correlations, diag=F) #turn into a upper triangle
correlations.upperTriangle<-correlations #take a copy of the original cor-mat
correlations.upperTriangle[!upperTriangle]<-NA#set everything not in upper triangle o NA
correlations_melted<-na.omit(melt(correlations.upperTriangle, value.name ="correlationCoef")) #use melt to reshape the matrix into triplets, na.omit to get rid of the NA rows
colnames(correlations_melted)<-c("X1", "X2", "correlation")


```


##Sampling map SNPs - grayscale

```
library(sp)  # classes for spatial data
library(raster)  # grids, rasters
library(rasterVis)  # raster visualisation
library(maptools)
library(rgeos)
# and their dependencies


CH_coords <-CH_coords_HiSeq
coordinates(CH_coords) <- c("Long.", "Lat.")  # set spatial coordinates
plot(CH_coords)
plot(CH_coords, pch = 20, cex = 1, col = "black")
crs.geo <- CRS("+proj=longlat +ellps=WGS84 +datum=WGS84")  # geographical, datum WGS84
proj4string(CH_coords) <- crs.geo  # define projection system of our data
summary(CH_coords)

require(spatial.tools)
elevation<-getData("alt", country="CH")
x <- terrain(elevation, opt = c("slope", "aspect"), unit = "degrees")
plot(x)
#getData("ISO3")
slope <- terrain(elevation, opt = "slope")
aspect <- terrain(elevation, opt = "aspect")
hill <- hillShade(slope, aspect, 40, 270)
#pdf("SERAD.pdf")
plot(hill, col = grey(0:100/100, alpha = 0.3), legend = FALSE, main = "CH")
#plot(elevation, col = rainbow(25, start=0, end=0.375, alpha = 0.5), main = "Sweden", add = TRUE)
#plot(CH_coords, pch = 20, cex = 1, col = "black", add = TRUE)
text(CH_coords, labels=CH_coords$Site, cex=0.6)
#dev.off()


```
