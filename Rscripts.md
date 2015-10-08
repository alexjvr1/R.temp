#Scripts that have not been fully optimised

##Tab 1

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
