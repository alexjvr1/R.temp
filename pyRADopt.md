#Optimisation of pyRAD thresholds: 

1. Clustering threshold

  Checks: cluster vs nucleotide diversity, vs 
  
2. MinDP

pyRAD uses the ML method from Lynch et al. 2008 to jointly estimate nucleotide heterozygosity and error for every base across a stack. .

This is estimated per stack within an individual to call homozygotes/ heterozygotes. 

The method seems robust from ~6-8x coverage, unless error rates are really high, or nucleotide diversity is very low. 


  Checks: nucleotide diversity vs depth, Ho vs He at different depths
  
  
3. Paralog filter??
  
  I'm currently filtering paralogs that are >2 SD from the mean depth. But I have such a high SD in my depth, that this doesn't seem like a very good filter. 
  
  I'm not sure how I can correct for this. 
  
  
##1. Clustering Threshold

Run pyRAD for a subset of individuals (a good representation of the genetic diversity in the population). 

If samples are overclustered, we expect higher nucleotide diversity & Heterozygosity. 

Underclustering will lead to more loci, but a drop-off in nucleotide diversity. 

Use the output from the stats/Pi_* file to create the plots. 




##2. Depth

UCLA groups found that the He/Ho tend more towards convergence when a higher minDP is used (i.e. minimum number of reads clustered across individuals.) 

To test whether my data show the same effect I will run minDP 6, 10, 20 initially. 

PyRAD needs to be run from s2. If I subset the samples that have already run (edits/ & clust.94/), it doesn't work. 


1. Compare nDepth vs nucleotide diversity (output from pyRAD) within and between pyRAD runs

2. As suggested (below) subset data to ~2Mil reads per individual, and rerun pyRAD. Compare nucleotide diversity between runs & to total depth. 

https://groups.google.com/forum/#!topic/pyrad-users/1VvKPnaSOHI


###SE

I'm using the 36 pyRADopt individuals. 


minDP 10 (gdcsrv1)

minDP 15 (gdcsrv2)


Copy s3. and s5. outputs from both minDP10 and minDP15 outputs from pyRAD into an excel file. 

Use R to draw some graphs; mainly to see the effect of depth on nulceotide diversity: 

![alt_txt][Fig1]
[Fig1]:https://cloud.githubusercontent.com/assets/12142475/15202838/abe31612-17b2-11e6-9f05-2b1930daae03.png

![alt_txt][Fig2]
[Fig2]:https://cloud.githubusercontent.com/assets/12142475/15202839/abe43ace-17b2-11e6-852c-561a6e6fc130.png

![alt_txt][Fig3]
[Fig3]:https://cloud.githubusercontent.com/assets/12142475/15202841/abee76d8-17b2-11e6-8c18-587137e1e1c3.png

![alt_txt][Fig4]
[Fig4]:https://cloud.githubusercontent.com/assets/12142475/15202840/abed663a-17b2-11e6-808c-9819f50e4cce.png

![alt_txt][Fig5]
[Fig5]:https://cloud.githubusercontent.com/assets/12142475/15202837/abe07006-17b2-11e6-9b21-feaebea44028.png




R code

This code plots MinDP10 and MinDP15 together on most graphs. The code for CH plots independent datasets. 
```
##pyRADopt: SE dataset - Final code
##plot polyfreq vs Depth minDP10

setwd("/Users/alexjvr/2016RADAnalysis/pyRADopt")

SE.MinDP <- read.csv("SE.Depth.n10.15.csv", header=T)
head(SE.MinDP)
summary(SE.MinDP)


#install.packages("ggplot2")
library(ggplot2)
library(reshape2)
library()

##########################################
##Depth vs polyfreq: SEMinDP10 and MinDP15

head(SE.MinDP)

p1 <- data.frame(x=SE.MinDP$d.9.me, y=SE.MinDP$poly.n10)

lm_eqn1 <- function(p1){
  m <- lm(y~x, p1);
  eq <- substitute(italic(y) == a + b %.% italic(x)*","~~italic(r)^2~"="~r2,
                   list(a = format(coef(m)[1], digits =2),
                        b = format(coef(m)[2], digits =2),
                        r2 = format(summary(m)$r.squared, digits =3)))
  as.character(as.expression(eq));
}

p2 <- data.frame(x=SE.MinDP$d.14.me, y=SE.MinDP$poly.n15)

lm_eqn2 <- function(p2){
  m <- lm(y~x, p2);
  eq <- substitute(italic(y) == a + b %.% italic(x)*","~~italic(r)^2~"="~r2,
                   list(a = format(coef(m)[1], digits =2),
                        b = format(coef(m)[2], digits =2),
                        r2 = format(summary(m)$r.squared, digits =3)))
  as.character(as.expression(eq));
}
lm_eqn2



zz <- melt(list(p1=p1, p2=p2), id.vars="x")
zz
p <- ggplot(zz, aes(x=x, y=value, color = L1)) +
  geom_point() + 
  geom_smooth(data=subset(zz, L1 %in% c("p1")), method="lm", se=T, color="black", formula= y~x) +
  geom_smooth(data=subset(zz, L1 %in% c("p2")), method="lm", se=T, color="black", formula= y~x) +
  scale_color_manual("Dataset", values =c("p1" = "darkgreen", "p2"="blue"), labels=c("MinDP10", "MinDP15"))
p

p <- p + xlab("Mean Sequencing Depth per Indiv") + ylab("Average Nucleotide diversity") + ggtitle("SE: Mean Depth x Nucleotide Diversity")
p

plot2 <- p + geom_text(size=3, x = 30, y = 0.0047, label = lm_eqn1(p1), colour="black", parse =T) + 
  geom_text(size=3, x = 45, y = 0.0047, label = lm_eqn2(p2), colour = "black", parse =T)

plot2


########################
########################

##Depth vs variance: SEMinDP10 and MinDP15

head(SE.MinDP)

p1 <- data.frame(x=SE.MinDP$d.9.me, y=SE.MinDP$d.9.sd)

lm_eqn1 <- function(p1){
  m <- lm(y~x, p1);
  eq <- substitute(italic(y) == a + b %.% italic(x)*","~~italic(r)^2~"="~r2,
                   list(a = format(coef(m)[1], digits =2),
                        b = format(coef(m)[2], digits =2),
                        r2 = format(summary(m)$r.squared, digits =3)))
  as.character(as.expression(eq));
}

p2 <- data.frame(x=SE.MinDP$d.14.me, y=SE.MinDP$d.14.sd)

lm_eqn2 <- function(p2){
  m <- lm(y~x, p2);
  eq <- substitute(italic(y) == a + b %.% italic(x)*","~~italic(r)^2~"="~r2,
                   list(a = format(coef(m)[1], digits =2),
                        b = format(coef(m)[2], digits =2),
                        r2 = format(summary(m)$r.squared, digits =3)))
  as.character(as.expression(eq));
}

zz <- melt(list(p1=p1, p2=p2), id.vars="x")
zz
p <- ggplot(zz, aes(x=x, y=value, color = L1)) +
  geom_point() + 
  geom_smooth(data=subset(zz, L1 %in% c("p1")), method="lm", se=T, color="black", formula= y~x) +
  geom_smooth(data=subset(zz, L1 %in% c("p2")), method="lm", se=T, color="black", formula= y~x) +
  scale_color_manual("Dataset", values =c("p1" = "darkgreen", "p2"="blue"), labels=c("MinDP10", "MinDP15"))
p

p <- p + xlab("Mean Sequencing Depth per Indiv") + ylab("Standard Dev") + ggtitle("SE: Mean Depth x SD of Depth")
p

plot2 <- p + geom_text(size=5, x = 45, y = 200, label = lm_eqn1(p1), colour="black", parse =T) + 
  geom_text(size=3, x = 30, y = 150, label = lm_eqn2(p2), colour = "black", parse =T)

plot2




##########################
##########################

##poly10 vs poly15

dfn10.n15 <- read.csv("SE.Depth.n10.15.csv")

df <- data.frame(x = dfn10.n15$poly.n10)
df$y <- dfn10.n15$poly.n15


p <- ggplot(data=df, aes(x=x, y=y)) +
  geom_smooth(method="lm", se=T, color="black", formula= y~x) +
  geom_point()
p

p <- p + xlab("nucleotide div (>9x)") + ylab("nucleotide div (>14x)") + ggtitle("SE: Nucleotide diversity per indiv at 10x vs 15x")
p


lm_eqn <- function(df){
  m <- lm(y~x, df);
  eq <- substitute(italic(y) == a + b %.% italic(x)*","~~italic(r)^2~"="~r2,
                   list(a = format(coef(m)[1], digits =2),
                        b = format(coef(m)[2], digits =2),
                        r2 = format(summary(m)$r.squared, digits =3)))
  as.character(as.expression(eq));
}

summary("m")

p1 <- p + geom_text(size=4, x = 0.0035, y = 0.0047, label = lm_eqn(df), parse =T) ##x & y = position on graph
p1


#########################################
#########################################
###Plot HWE from plink generated output
##I want to plot per locus He vs Ho for n10 and n15 on the same plot

##Generated data in Plink (--hardy)
##output is *.hwe, with several columns. 
##2/SNP, 3/TEST (filter for ALL), 6/Genotype count, 7/Ho, 8/He, 9/P-value for HWE


setwd("/Users/alexjvr/2016RADAnalysis/pyRADopt/SE.tests/")
df.n10 <- read.table("SE.DP10.hwe", header = T)
summary(df.n10)
head(df.n10)


df.n10.v2 <- subset(df.n10, TEST=="ALL", select=SNP:E.HET.)
head(df.n10.v2)
summary(df.n10.v2)

library(ggplot2)

df.n10.3 <- df.n10.v2[with(df.n10.v2, order(E.HET.)),]##sort by E.HET
head(df.n10.3)

df.n10.3$Nr <- c(1:24825)##Add column numbered 1:22646 for the sorted SNPs
head(df.n10.3)

df.n15 <- read.table("SE.DP15.hwe", header = T)
summary(df.n15)
head(df.n15)


df.n15.v2 <- subset(df.n15, TEST=="ALL", select=SNP:E.HET.)
head(df.n15.v2)
summary(df.n15.v2)

library(ggplot2)

df.n15.3 <- df.n15.v2[with(df.n15.v2, order(E.HET.)),]##sort by E.HET
head(df.n15.3)

df.n15.3$Nr <- c(1:13044)##Add column numbered 1:22646 for the sorted SNPs
head(df.n15.3)

##plot O.HET & E.HET vs SNPnr

#MinDP10 
library(ggplot2)

df1<-data.frame(x=df.n10.3$Nr,y=df.n10.3$O.HET.)
df2<-data.frame(x=df.n10.3$Nr,y=df.n10.3$E.HET.)

ggplot(df1,aes(x,y))+geom_point(aes(color="O.HET."))+
  geom_point(data=df2,aes(color="E.HET"))+
  labs(color="Key") +
  ylab("Heterozygosity") +
  xlab("variant") +
  ggtitle("SE_MinDP10: Observed vs Expected Het")

#MinDP15
library(ggplot2)

df1<-data.frame(x=df.n15.3$Nr,y=df.n15.3$O.HET.)
df2<-data.frame(x=df.n15.3$Nr,y=df.n15.3$E.HET.)

ggplot(df1,aes(x,y))+geom_point(aes(color="O.HET."))+
  geom_point(data=df2,aes(color="E.HET"))+
  labs(color="Key") +
  ylab("Heterozygosity") +
  xlab("variant") +
  ggtitle("SE_MinDP15: Observed vs Expected Het")


###########################################  
###Plot He vs Ho frequencies

library(ggplot2)
library(reshape2)

HET10 <- data.frame(x=df.n10.3$O.HET., y=df.n10.3$E.HET.)
HET15 <- data.frame(x=df.n15.3$O.HET., y=df.n15.3$E.HET.)
head(HET10)

HETm10 <- melt(HET10, variable.name = "Obs.Exp", value.name = "MinDP10Het_value")
HETm15 <- melt(HET15, variable.name = "Obs.Exp", value.name = "MinDP15Het_value")
#head(HETm15)

#HETall <- merge(HETm10, HETm15, by="Obs.Exp")

plot.freq <- ggplot(HET10) +
  geom_freqpoly(data=HETm10, aes(x = MinDP10Het_value, lty= Obs.Exp), binwidth=0.01, size=1, colour="darkgreen") +
  geom_freqpoly(data=HETm15, aes(x = MinDP15Het_value, lty= Obs.Exp), binwidth=0.01, size=1, colour = "blue") +
  ylab("Frequency") +
  xlab("Heterozygosity") +
  ggtitle("SE: Frequency of Obs vs Expected Het")

plot.freq

#code for Key doesn't work
#scale_colour_discrete(name = "Dataset", breaks = c("HETm10$value", "HETm15$value"),
#                      labels=c("minDP10", "minDP15")) +
# scale_linetype_discrete(name = "Heterozygosity", breaks = c("HETm10$variable", "HETm15$variable"), 
#                        labels=c("O.HET", "E.HET"))

#values =c("HETm10" = "darkgreen", "HETm15"="blue"), labels=c("MinDP10", "MinDP15"))


```



###CH

I'm using 55 individuals randomly chosen from the FstQst sample folder. 

I've transferred these to Euler & will run MinDP10, MinDP15, MinDP20 for these

**These ran on GDCsrv. and vcftools/bcftools will be run on the mac. 
/Users/alexjvr/2016RADAnalysis/pyRADopt/input

Plot: 

1. average depth per individual vs nucleotide diversity

2. Average depth per site vs nucleotide diversity
 
This isn't possible, because we can't get per locus depth estimates easily from pyRAD
```
vcftools --vcf CHDepthtest.MinDP10.vcf --site-mean-depth --recode-INFO-all  ##this information is not in the vcf file! I guess it gets removed by pyRAD, which only outputs the SNP information... 

vcftools --vcf CHDepthtest.MinDP10.vcf --site-pi --recode-INFO-all
```

Write column 3 to a new file called freq.pi. Move this to gdcsrv to draw a histogram in bash: 

/gdc_home4/alexjvr/DepthTest/CH.plots


```
cat out.sites.pi | awk '{print $3}' > freq.pi

scp freq.pi alexjvr@gdcsrv1.ethz.ch:/gdc_home4/alexjvr/DepthTest/CH.plots/


gnuplot << \EOF 
set terminal dumb size 120, 30
set autoscale 
unset label
set title "Histogram of nucleotide diversity per locus"
set ylabel "Number of Occurrences"
set xlabel "nucleotide diversity"
binwidth=0.01
bin(x,width)=width*floor(x/width) + binwidth/2.0
plot 'freq.pi' using (bin( $1,binwidth)):(1.0) smooth freq with boxes
pause -1
EOF
```

MinDP10 

![alt_txt][CH.DP10_site-pi]
[CH.DP10_site-pi]:https://cloud.githubusercontent.com/assets/12142475/15096083/694e993c-149e-11e6-8934-cf5ffb21feb1.png


MinDP15

![alt_txt][CH.DP15_site-pi]
[CH.DP15_site-pi]:https://cloud.githubusercontent.com/assets/12142475/15096085/6f2191d4-149e-11e6-9170-ab624d220d50.png



The good news is that the nucleotide frequency distributions look exactly the same at both depths. 

(Note the number on the side to show the difference in the number of loci) 

But, there are some sites that have a very high nucleotide diversity: ~1.0. Clearly this is from incorrectly clustered loci. I could either filter these out by setting a parameter in pyRAD, or after the fact, with VCFtools. 

The amount of missing data per individual for the unfiltered dataset: 

DP10

![alt_txt][CH.DP10_missing]
[CH.DP10_missing]:https://cloud.githubusercontent.com/assets/12142475/15125545/4f5eef6e-15e2-11e6-8bc9-871283b67267.png

DP15

![alt_txt][CH.DP15_missing]
[CH.DP15_missing]:https://cloud.githubusercontent.com/assets/12142475/15125550/503a3768-15e2-11e6-9942-fad8a57c836c.png


Since all the pyRAD runs have been completed, I will do this after the fact: 

Filter for MAC 3

```
vcftools --vcf CHDepthtest.MinDP10.vcf --mac 3 --recode --recode-INFO-all --out s1.CH.DP10

VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf CHDepthtest.MinDP10.vcf
	--recode-INFO-all
	--mac 3
	--out s1.CH.DP10
	--recode

Eighth Header entry should be INFO: INFO    
After filtering, kept 55 out of 55 Individuals
Outputting VCF file...
After filtering, kept 115033 out of a possible 301652 Sites
Run Time = 12.00 seconds
```

and draw the graphs


```
vcftools --vcf s1.CH.DP10.recode.vcf --missing-indv

VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf s1.CH.DP10.recode.vcf
	--missing-indv

After filtering, kept 55 out of 55 Individuals
Outputting Individual Missingness
After filtering, kept 115033 out of a possible 115033 Sites
Run Time = 2.00 seconds

mawk '!/IN/' out.imiss | cut -f5 > totalmissing
gnuplot << \EOF 
set terminal dumb size 120, 30
set autoscale 
unset label
set title "Histogram of % missing data per individual"
set ylabel "Number of Occurrences"
set xlabel "% of missing data"
#set yr [0:100000]
binwidth=0.01
bin(x,width)=width*floor(x/width) + binwidth/2.0
plot 'totalmissing' using (bin( $1,binwidth)):(1.0) smooth freq with boxes
pause -1
EOF
```

![alt_txt][CH.DP10.mac3]
[CH.DP10.mac3]:https://cloud.githubusercontent.com/assets/12142475/15125546/4f848828-15e2-11e6-9f02-5c2618828071.png

```

vcftools --vcf CHDepthtest.MinDP10.vcf --max-missing 0.5 --recode --recode-INFO-all --out s2CH.DP10

VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf CHDepthtest.MinDP10.vcf
	--recode-INFO-all
	--max-missing 0.5
	--out s2CH.DP10
	--recode

Eighth Header entry should be INFO: INFO    
After filtering, kept 55 out of 55 Individuals
Outputting VCF file...
After filtering, kept 23077 out of a possible 301652 Sites
Run Time = 8.00 seconds
```



![alt_txt][CH.DP10.maxmissing0.5]
[CH.DP10.maxmissing0.5]:https://cloud.githubusercontent.com/assets/12142475/15125549/501e6790-15e2-11e6-9a7b-ea867c1a5427.png



###Individual-level plots

These are based on the s3* and s5* outputs from pyRAD (/stats). 

I copied these data to .csv file on the mac and created the following plots in R: 

** The plots are created with the scripts from SE, but for individual datasets the CH scripts can be used (below)


![alt_txt][Fig1]
[Fig1]:https://cloud.githubusercontent.com/assets/12142475/15226301/070cb8a2-1837-11e6-8407-a67ec9f18dea.png

![alt_txt][Fig2]
[Fig2]:https://cloud.githubusercontent.com/assets/12142475/15226298/070511d8-1837-11e6-9f24-d4c187b8d646.png

![alt_txt][Fig3]
[Fig3]:https://cloud.githubusercontent.com/assets/12142475/15226297/07047336-1837-11e6-801b-262064cbebfe.png

![alt_txt][Fig4]
[Fig4]:https://cloud.githubusercontent.com/assets/12142475/15226299/0707d846-1837-11e6-8253-41f20618f5a3.png

![alt_txt][Fig5]
[Fig5]:https://cloud.githubusercontent.com/assets/12142475/15226302/070d9286-1837-11e6-8c0f-bd908c7832a9.png

![alt_txt][Fig]
[Fig6]:https://cloud.githubusercontent.com/assets/12142475/15226300/070a867c-1837-11e6-8bc5-8f06db1b2fd5.png







R-code 
```
##pyRADopt
##plot polyfreq vs Depth minDP10

setwd("/Users/alexjvr/2016RADAnalysis/pyRADopt")

CH.MinDP10 <- read.csv("CH.Depth.n10.csv", header=T)
head(CH.MinDP10)
summary(CH.MinDP10)

##this qplot works
#qplot(x = CH.MinDP10$d.9.me, y = CH.MinDP10$poly, data=CH.MinDP10, geom=c("point", "smooth"), method="lm", formula=y~x, xlab="avg mean depth >9x per indiv", ylab="avg nucleotide diversity per indiv")

#install.packages("ggplot2")
library(ggplot2)

df <- data.frame(x = CH.MinDP10$d.9.me)
df$y <- CH.MinDP10$poly

?ggplot

p <- ggplot(data=df, aes(x=x, y=y)) +
  geom_smooth(method="lm", se=F, color="black", formula= y~x) +
  geom_point()
p

p <- p + xlab("average depth per indiv (>9x)") + ylab("average nucleotide diversity per indiv") + ggtitle("CH.DP10: mean depth x polyfreq")
p


lm_eqn <- function(df){
  m <- lm(y~x, df);
  eq <- substitute(italic(y) == a + b %.% italic(x)*","~~italic(r)^2~"="~r2,
                   list(a = format(coef(m)[1], digits =2),
                        b = format(coef(m)[2], digits =2),
                        r2 = format(summary(m)$r.squared, digits =3)))
  as.character(as.expression(eq));
}

summary(m)

p1 <- p + geom_text(size=7, x = 30, y = 0.0054, label = lm_eqn(df), parse =T) ##x & y = position on graph
p1



##limiting the x or y axis will remove some of the datapoints. But the df needs to be re-defined to calculate the regression formula. 
p3 <- p1 + xlim(25, 35) 
p3


##CH.DP10: mean depth x polyfreq (DP >25)

#subset the data

CH.DP10.24x <- subset(CH.MinDP10, d.9.me >= 25, 
                      select=c("d.9.me", "poly"))

summary(CH.DP10.24x)

df <- data.frame(x = CH.DP10.24x$d.9.me)
df$y <- CH.DP10.24x$poly


p <- ggplot(data=df, aes(x=x, y=y)) +
  geom_smooth(method="lm", se=F, color="black", formula= y~x) +
  geom_point()
p

p <- p + xlab("average depth per indiv (>9x)") + ylab("average nucleotide diversity per indiv") + ggtitle("CH.DP10: mean depth (>25x) x polyfreq")
p


lm_eqn <- function(df){
  m <- lm(y~x, df);
  eq <- substitute(italic(y) == a + b %.% italic(x)*","~~italic(r)^2~"="~r2,
                   list(a = format(coef(m)[1], digits =2),
                        b = format(coef(m)[2], digits =2),
                        r2 = format(summary(m)$r.squared, digits =3)))
  as.character(as.expression(eq));
}

summary(m)

p1 <- p + geom_text(size=7, x = 32.5, y = 0.0047, label = lm_eqn(df), parse =T) ##x & y = position on graph
p1

##Relationship between Depth & Variance?

df <- data.frame(x=CH.MinDP10$d.9.me)
df$y <- CH.MinDP10$d.9.sd

p <- ggplot(data=df, aes(x=x, y=y)) +
  geom_smooth(method="lm", se=F, color="black", formula= y~x) +
  geom_point()
p

p <- p + xlab("average depth per indiv (>9x)") + ylab("Standard Dev") + ggtitle("CH.DP10: mean depth (>9x) x SD of Depth")
p


lm_eqn <- function(df){
  m <- lm(y~x, df);
  eq <- substitute(italic(y) == a + b %.% italic(x)*","~~italic(r)^2~"="~r2,
                   list(a = format(coef(m)[1], digits =2),
                        b = format(coef(m)[2], digits =2),
                        r2 = format(summary(m)$r.squared, digits =3)))
  as.character(as.expression(eq));
}

summary(m)

p1 <- p + geom_text(size=7, x = 25, y = 120, label = lm_eqn(df), parse =T) ##x & y = position on graph
p1

###############################
###############################
##And plot minDP15

CH.MinDP15 <- read.csv("CH.Depth.n15.csv", header=T)
head(CH.MinDP15)
summary(CH.MinDP15)

df <- data.frame(x = CH.MinDP15$d.14.me)
df$y <- CH.MinDP15$poly


p <- ggplot(data=df, aes(x=x, y=y)) +
  geom_smooth(method="lm", se=F, color="black", formula= y~x) +
  geom_point()
p

p <- p + xlab("average depth per indiv (>14x)") + ylab("average nucleotide diversity per indiv") + ggtitle("CH.DP15: mean depth x polyfreq")
p


lm_eqn <- function(df){
  m <- lm(y~x, df);
  eq <- substitute(italic(y) == a + b %.% italic(x)*","~~italic(r)^2~"="~r2,
                   list(a = format(coef(m)[1], digits =2),
                        b = format(coef(m)[2], digits =2),
                        r2 = format(summary(m)$r.squared, digits =3)))
  as.character(as.expression(eq));
}

summary(m)

p1 <- p + geom_text(size=7, x = 45, y = 0.0055, label = lm_eqn(df), parse =T) ##x & y = position on graph
p1



##limiting the x or y axis will remove some of the datapoints. But the df needs to be re-defined to calculate the regression formula. 
p3 <- p1 + xlim(25, 35) 
p3

##########################
##########################

##poly10 vs poly15

dfn10.n15 <- read.csv("CH.Depth.n10.15.csv")

df <- df <- data.frame(x = dfn10.n15$poly.n10)
df$y <- dfn10.n15$poly.n15


p <- ggplot(data=df, aes(x=x, y=y)) +
  geom_smooth(method="lm", se=F, color="black", formula= y~x) +
  geom_point()
p

p <- p + xlab("nucleotide div (>9x)") + ylab("nucleotide div (>14x)") + ggtitle("Nucleotide diversity per indiv at 10x vs 15x")
p


lm_eqn <- function(df){
  m <- lm(y~x, df);
  eq <- substitute(italic(y) == a + b %.% italic(x)*","~~italic(r)^2~"="~r2,
                   list(a = format(coef(m)[1], digits =2),
                        b = format(coef(m)[2], digits =2),
                        r2 = format(summary(m)$r.squared, digits =3)))
  as.character(as.expression(eq));
}

summary(m)

p1 <- p + geom_text(size=7, x = 0.0044, y = 0.0057, label = lm_eqn(df), parse =T) ##x & y = position on graph
p1


##Relationship between Depth & Variance at n15?

df <- data.frame(x=CH.MinDP15$d.14.me)
df$y <- CH.MinDP15$d.14.sd

p <- ggplot(data=df, aes(x=x, y=y)) +
  geom_smooth(method="lm", se=F, color="black", formula= y~x) +
  geom_point()
p

p <- p + xlab("average depth per indiv (>14x)") + ylab("Standard Dev") + ggtitle("CH.DP15: mean depth (>14x) x SD of Depth")
p


lm_eqn <- function(df){
  m <- lm(y~x, df);
  eq <- substitute(italic(y) == a + b %.% italic(x)*","~~italic(r)^2~"="~r2,
                   list(a = format(coef(m)[1], digits =2),
                        b = format(coef(m)[2], digits =2),
                        r2 = format(summary(m)$r.squared, digits =3)))
  as.character(as.expression(eq));
}

summary(m)

p1 <- p + geom_text(size=7, x = 37, y = 150, label = lm_eqn(df), parse =T) ##x & y = position on graph
p1
```



To check how much the depth filter makes a difference in pyRAD: 



- It's not very easy to get the per locus depth estimates from pyRAD. The per individual counts are collapsed in the /cluster files. The concensus sequences from each individual are then clustered across individuals. 


compare avg depth per site vs nucleotide diversity for n10 dataset, filtered for n15 coverage, to pyRAD n15 coverage
```

```


4. Average depth per site vs Heterozygosity
```

```

5. Ho vs He at Depth10 vs Depth 15

O.Het varies greatly at different E.Het. I'm concerned about the number of SNPs with Ho > 0.5, but not sure how to sensibly filter for this. 
 
 
MinDP10

![alt_txt][He.Ho.10]
[He.Ho.10]:https://cloud.githubusercontent.com/assets/12142475/15153135/f5375d5e-168c-11e6-9c3f-61639d383ec4.png

 

MinDP15

![alt_txt][He.Ho.15]
[He.Ho.15]:https://cloud.githubusercontent.com/assets/12142475/15153134/f5337932-168c-11e6-96ca-308040490932.png





Derive Ho & He using Plink

1. Filter VCF file for MAC & max-missing

2. Convert to plink

3. use Plink to obtain HWE output

4. copy to mac & use R to plot graphs

```
vcftools --vcf CHDepthtest.MinDP15.vcf --mac 3 --recode --recode-INFO-all --out s1.CH.DP15

VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf CHDepthtest.MinDP15.vcf
	--recode-INFO-all
	--mac 3
	--out s1.CH.DP15
	--recode

Eighth Header entry should be INFO: INFO    
After filtering, kept 55 out of 55 Individuals
Outputting VCF file...
After filtering, kept 57074 out of a possible 154042 Sites
Run Time = 6.00 seconds


vcftools --vcf s1.CH.DP15.recode.vcf --max-missing 0.5 --recode --recode-INFO-all --out s2.CH.DP15

VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf s1.CH.DP15.recode.vcf
	--recode-INFO-all
	--max-missing 0.5
	--out s2.CH.DP15
	--recode

After filtering, kept 55 out of 55 Individuals
Outputting VCF file...
After filtering, kept 7918 out of a possible 57074 Sites
Run Time = 2.00 seconds


vcftools --vcf s2.CH.DP15.recode.vcf --plink --out HWE/s2.CH.DP15

VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf s2.CH.DP15.recode.vcf
	--out HWE/s2.CH.DP15
	--plink

After filtering, kept 55 out of 55 Individuals
Writing PLINK PED and MAP files ... 
	PLINK: Only outputting biallelic loci.
Done.
After filtering, kept 7918 out of a possible 7918 Sites
Run Time = 0.00 seconds

plink --file s2CHDP10 --hardy

@----------------------------------------------------------@
|        PLINK!       |     v1.07      |   10/Aug/2009     |
|----------------------------------------------------------|
|  (C) 2009 Shaun Purcell, GNU General Public License, v2  |
|----------------------------------------------------------|
|  For documentation, citation & bug-report instructions:  |
|        http://pngu.mgh.harvard.edu/purcell/plink/        |
@----------------------------------------------------------@

Web-based version check ( --noweb to skip )
Connecting to web...  OK, v1.07 is current

+++ PLINK 1.9 is now available! See above website for details +++ 

Writing this text to log file [ plink.log ]
Analysis started: Tue May 10 05:19:24 2016

Options in effect:
	--file s2CHDP10
	--hardy

22652 (of 22652) markers to be included from [ s2CHDP10.map ]
Warning, found 55 individuals with ambiguous sex codes
These individuals will be set to missing ( or use --allow-no-sex )
Writing list of these individuals to [ plink.nosex ]
55 individuals read from [ s2CHDP10.ped ] 
0 individuals with nonmissing phenotypes
Assuming a disease phenotype (1=unaff, 2=aff, 0=miss)
Missing phenotype value is also -9
0 cases, 0 controls and 55 missing
0 males, 0 females, and 55 of unspecified sex
Before frequency and genotyping pruning, there are 22652 SNPs
55 founders and 0 non-founders found
Writing Hardy-Weinberg tests (founders-only) to [ plink.hwe ] 
2180 markers to be excluded based on HWE test ( p <= 0.001 )
	0 markers failed HWE test in cases
	0 markers failed HWE test in controls
Total genotyping rate in remaining individuals is 0.668754
0 SNPs failed missingness test ( GENO > 1 )
0 SNPs failed frequency test ( MAF < 0 )
After frequency and genotyping pruning, there are 20472 SNPs
After filtering, 0 cases, 0 controls and 55 missing
After filtering, 0 males, 0 females, and 55 of unspecified sex
```

Plot in R:
```
###Plot HWE from plink generated output
##I want to plot per locus He vs Ho for n10 and n15 on the same plot

##Generated data in Plink (--hardy)
##output is *.hwe, with several columns. 
##2/SNP, 3/TEST (filter for ALL), 6/Genotype count, 7/Ho, 8/He, 9/P-value for HWE


setwd("/Users/alexjvr/2016RADAnalysis/pyRADopt/input/HWE")
df.n10 <- read.table("CHDP10.plink.hwe", header = T)
summary(df.n10)
head(df.n10)


df.n10.v2 <- subset(df.n10, TEST=="ALL", select=SNP:E.HET.)
head(df.n10.v2)
summary(df.n10.v2)

library(ggplot2)

df.n10.3 <- df.n10.v2[with(df.n10.v2, order(E.HET.)),]##sort by E.HET
head(df.n10.3)

df.n10.3$Nr <- c(1:22652)##Add column numbered 1:22646 for the sorted SNPs
head(df.n10.3)

##plot O.HET & E.HET vs SNPnr

library(ggplot2)

df1<-data.frame(x=df.n10.3$Nr,y=df.n10.3$O.HET.)
df2<-data.frame(x=df.n10.3$Nr,y=df.n10.3$E.HET.)

ggplot(df1,aes(x,y))+geom_line(aes(color="O.HET."))+
  geom_line(data=df2,aes(color="E.HET"))+
  labs(color="Obs vs Exp Het")


##################
#######Same for CH.DP15


df.n15 <- read.table("CHDP15.hwe", header = T)
summary(df.n15)
head(df.n15)


df.n15.v2 <- subset(df.n15, TEST=="ALL", select=SNP:E.HET.)
head(df.n15.v2)
summary(df.n15.v2)

library(ggplot2)

df.n15.3 <- df.n15.v2[with(df.n15.v2, order(E.HET.)),]##sort by E.HET
head(df.n15.3)

df.n15.3$Nr <- c(1:7830)##Add column numbered 1:22646 for the sorted SNPs
head(df.n15.3)

##plot O.HET & E.HET vs SNPnr

library(ggplot2)

df1<-data.frame(x=df.n15.3$Nr,y=df.n15.3$O.HET.)
df2<-data.frame(x=df.n15.3$Nr,y=df.n15.3$E.HET.)

ggplot(df1,aes(x,y))+geom_line(aes(color="O.HET."))+
  geom_line(data=df2,aes(color="E.HET"))+
  labs(color="Obs vs Exp Het, CH.MinDP15")


#############################
##plot O.HET vs E.HET

p2 <- ggplot(df.n15.3, aes(x=df.n15.3$E.HET.,y=df.n15.3$O.HET.)) + geom_line() + ylab("E.HET.") + xlab("O.HET")
p2


                                                     
```
 






###BV

49 individuals from demultiplexedBV2and3

I've transferred these to Euler & will run MinDP10, MinDP15, MinDP20 for these



##Submitting jobs to Euler

```
ssh alexjvr@gdcsrv1.ethz.ch
jk.W3.mm
ssh alexjvr@euler.ethz.ch
W465.Smq

cp pyrad.run.cmds /cluster/home/alexjvr/ /cluster/scratch/alexjvr/*file*

#adjust parameters in cmds file

  - time for each run
  
  - # out any commands you don't want run

nano params.txt

#adjust any parameters

./pyrad.run.cmds
```


##See Stefan's email explaining all of this - in the flagged folder in my inbox. 

```
Hi Alex

Below you find the how-to for running pyrad on Euler. It includes the installation of the "package", what to copy where and information on how to actually start a job. 
Attached you find the files that are mentioned below. 
First you have to copy these files to our servers, e.g. gdcsrv1

1) on gdcsrv1, copy the pyrad package to euler (please note the colon at the end)
    scp pyrad-3.0.63.tar  alexjvr@euler.ethz.ch:
   do the same for the pyrad.run.cmds
    scp pyrad.run.cmds  alexjvr@euler.ethz.ch:
2) login to euler
3) create a bin directory (unless already present)
    mkdir bin
4) now move the package to bin and change into it
    mv pyrad-3.0.63.tar bin/
    cd bin/
6) extract the package
    tar xvf pyrad-3.0.63.tar
7) change to the home again
    cd ..
8) Use the command-line editor nano to open the file .bashrc
    nano .bashrc
   now enter the following line after # User specific aliases and functions:
    export PATH=/cluster/home/alexjvr/bin/pyrad-3.0.63/pyrad/:$PATH
   To save and exit: 
   press control-o, then hit return, then control-x
9) now test if pyrad is available
    source .bashrc
    which pyrad 
10) now move to your scratch space and create a run directory and change into it
     cd /cluster/scratch/alexjvr/
     mkdir pyrad_1
     cd pyrad_1 
11) copy the pyrad.run.cmds from your home to the current location
     cp ~/pyrad.run.cmds . 
12) adjust the settings in the pyrad.run.cmds. Especially important is the 
    memory and the run time (wall-clock-time).
     There is for example: 
     -n the number of CPUs. You probably always want 24
     -W for wall-clock-time. The job will run for at 
     most that long (-> will be put into scheduler queue according to this)
     adjust this according to my estimates. Anything longer than 120 hours goes
     into the 30d queue and might then wait for several hours before starting. 
     -R "span[hosts=1]", makes sure it runs on one node.
     -R "rusage[mem=500]", for reserving memory. This is per CPU, so in our 
     case multiplied by 24. If you want all memory on a 256GB node, then put
     mem=10500
    Usually, I make sure to give a job enough resources, because otherwise it 
    might get killed by the scheduler prematurely. On the other hand do not 
    request too much, because then it might slip into a "higher" queue.
     -J "step2", gives the command a name, which can be read by the next 
     -w "ended(step2)", wait until command "step2" is done
     And finally of course the pyrad commands, e.g.:pyrad -p params.txt -s 2
    Note: you can out-comment a command with #, like I did with step 1,6,7
13) make the pyrad.run.cmds file executable:
     chmod 755 pyrad.run.cmds
14) adjust your params.txt file, especially the location of the input files. 
15) start the pyrad job:
    ./pyrad.run.cmds 
    This should print a couple of 
        Generic job.
        Job <11204117> is submitted to queue ...
16) now you can verify that the jobs are in the queue and/or running:
     bjobs
17) once a subjob is done, or an error occurred, you get an lsf.o.... file 
    In there you will find info about the jobs resource usage and status. 

OK, that was a long how-to, I hope all is clear and will work. Otherwise, let me
know and I will try to help. Or you could come to the GDC and we could start the 
first jobs together. 
```


