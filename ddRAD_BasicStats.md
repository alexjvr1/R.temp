##SNP filtering

after the basic SNP filters, I need to filter the dataset more for all the basic stats, to get the best representation of the data. 

Also to reduce the dataset so that the analyses don't take too long. 

#Basic Statistics

For an initial look at the data, summary stats, and population genetics. 

I will mainly use adegenet and resources specified on their website: 

http://adegenet.r-forge.r-project.org/

##Data conversion

pgdSpider can do most conversions. 
```
/Users/alexjvr/Software/PGDSpider/PGDSpider_2.0.8.2
java -Xmx1024m -Xms512m -jar PGDSpider2.jar
```

Working through the following tutorial: Analysing Genome wide SNPs using AdeGenet 
http://adegenet.r-forge.r-project.org/files/tutorial-genomics.pdf

##Note on sensitivity analyses

Both to get to know the data set and how the different analyses react to different datasets, it is good practice to rerun the basic statistics using different SNP filters. 

The filters that are probably the most influential are: 

1. individual genotyping rate
2. locus genotyping rate
3. Minor allele frequency (maf)

###Fst
Apparently Fst estimates and PCA are both really sensitive to which SNPs end up in the final data set. Some (e.g. Roesti et al. 2012), suggest a MAF of no less than 0.25. 

See this paper from Rasmus Nielsen's group: 

http://www.genetics.org/content/early/2013/08/15/genetics.113.154740.full.pdf

I will test how influential this is by running the following filters: 

1. MAF 0.1
2. MAF 0.25
3. MAF 0.3

vcf code: 
```
vcftools --vcf subset.final.vcf --maf 0.25 --recode --recode-INFO-all --out subset.final.maf25
```

For the final dataset (filter: 50% genotyping rate, 10% missing data per locus, MAC 3, min depth 3) = 16478 loci, 137 indivs, the filters yield the following: 

1. MAF 0.10 -> 11992, 137 indivs 
2. MAF 0.25 -> 5978, 137 indivs
3. MAF 0.30 -> 4496, 137 indivs


Average Fst estimates (from GenPop on the web): 

Pops
![alt_txt][Pop]
[Pop]:https://cloud.githubusercontent.com/assets/12142475/9013257/16851c82-37bb-11e5-8039-e2784ff7f2c6.png

MAF10
![alt_txt][MAF10]
[MAF10]:https://cloud.githubusercontent.com/assets/12142475/9013262/1e165736-37bb-11e5-98fd-6b14839f4649.png

MAF25
![alt_txt][MAF25]
[MAF25]:https://cloud.githubusercontent.com/assets/12142475/9013269/24395410-37bb-11e5-8483-0673b18804b2.png

MAF30
![alt_txt][MAF30]
[MAF30]:https://cloud.githubusercontent.com/assets/12142475/9013277/2d6f4152-37bb-11e5-9eb3-891c9c74816f.png


And for the full CH dataset (19 Oct) 


```
vcftools --vcf subset.final.vcf --maf 0.25 --recode --recode-INFO-all --out subset.final.maf25
```

1. MAF 0.10 -> 72894 out of a possible 205921 loci, 436 indivs 
2. MAF 0.25 -> 34747 out of a possible 205921, 436 indivs
3. MAF 0.30 -> 26482 out of a possible 205921, 436 indivs


Fst

I need to first create a file for each population with all the individual names in the file. 

-> in total 58 populations ranging in size from 1 (gdwe) -20 indivs. 

And then code all the between population comparisions for Fst
```
vcftools --vcf MAFtest/CH530maf0.1.recode.vcf --weir-fst-pop abnd_pop --weir-fst-pop alpl_pop --weir-fst-pop apla_pop --weir-fst-pop bach_pop --weir-fst-pop bela_pop --weir-fst-pop bend_pop --weir-fst-pop birk_pop --weir-fst-pop bnnp_pop --weir-fst-pop boty_pop --weir-fst-pop buel_pop --weir-fst-pop cava_pop --weir-fst-pop csee_pop --weir-fst-pop egel_pop --weir-fst-pop ente_pop --weir-fst-pop fess_pop --weir-fst-pop flor_pop --weir-fst-pop forn_pop --weir-fst-pop full_pop --weir-fst-pop fuor_pop --weir-fst-pop gdwe_pop --weir-fst-pop gott_pop --weir-fst-pop grma_pop --weir-fst-pop grsh_pop --weir-fst-pop gruu_pop --weir-fst-pop hdns_pop --weir-fst-pop laun_pop --weir-fst-pop lucm_pop --weir-fst-pop magn_pop --weir-fst-pop mart_pop --weir-fst-pop mctn_pop --weir-fst-pop mgns_pop --weir-fst-pop moal_pop --weir-fst-pop moir_pop --weir-fst-pop muet_pop --weir-fst-pop munt_pop --weir-fst-pop oalp_pop --weir-fst-pop orge_pop --weir-fst-pop pizo_pop --weir-fst-pop pozz_pop --weir-fst-pop rade_pop --weir-fst-pop roes_pop --weir-fst-pop rotc_pop --weir-fst-pop sali_pop --weir-fst-pop scai_pop --weir-fst-pop seeo_pop --weir-fst-pop seji_pop --weir-fst-pop span_pop --weir-fst-pop stba_pop --weir-fst-pop stls_pop --weir-fst-pop tals_pop --weir-fst-pop tamv_pop --weir-fst-pop tana_pop --weir-fst-pop trim_pop  --weir-fst-pop trpa_pop --weir-fst-pop vilt_pop --weir-fst-pop vora_pop --weir-fst-pop wdji_pop --weir-fst-pop zeni_pop --out MAFtest/fst.maf.01
```

1. MAF 0.10 -> 72894 out of a possible 205921, 436 indivs -> WCglobal = 0.21745; WCweighted = 0.22775 
2. MAF 0.25 -> 34747 out of a possible 205921, 436 indivs -> WCglobal = 0.25245; WCweighted = 0.25353
3. MAF 0.30 -> 26482 out of a possible 205921, 436 indivs -> WCglobal = 0.25762; WCweighted = 0.25799


So I will use the MAF 0.25 dataset. 


Next: 

1. Genetic diversity - CH, SE
                     - per population
                     - CHN, CHS
2. Pairwise Fst (hierfstat)- CHall, SEall
            - CHN, CHS
            - Across elevation
3. PCA for all of the above
4. Unrooted dendrogram for all
5. 

Convert data to plink
```
vcftools --vcf CH436.final.vcf --out /gdc_home4/alexjvr/CH530SNPfiltering/filteredforPCA/CH436.final --plink
plink --file CH436.final --out CH436.final.plink --recodeA



Options in effect:
	--file CH436.final
	--out CH436.final.plink
	--recodeA

** For gPLINK compatibility, do not use '.' in --out **
34700 (of 34700) markers to be included from [ CH436.final.map ]
Warning, found 436 individuals with ambiguous sex codes
Writing list of these individuals to [ CH436.final.plink.nosex ]
436 individuals read from [ CH436.final.ped ] 
0 individuals with nonmissing phenotypes
Assuming a disease phenotype (1=unaff, 2=aff, 0=miss)
Missing phenotype value is also -9
0 cases, 0 controls and 436 missing
0 males, 0 females, and 436 of unspecified sex
Before frequency and genotyping pruning, there are 34700 SNPs
436 founders and 0 non-founders found
Total genotyping rate in remaining individuals is 0.379948
0 SNPs failed missingness test ( GENO > 1 )
0 SNPs failed frequency test ( MAF < 0 )
After frequency and genotyping pruning, there are 34700 SNPs
After filtering, 0 cases, 0 controls and 436 missing
After filtering, 0 males, 0 females, and 436 of unspecified sex
Writing recoded file to [ CH436.final.plink.raw ] 

```

436 individuals

34700 loci

This is a huge amount of data. So the best is to subsample this down to 5000-10 000 loci, otherwise the analyses just take too long to run: 

This can be done in PLINK
```
plink --file --thin-count 5000 --out CH436.5000.final
```
this only works in Plink 1.9, so no idea how to fix this. 

In stead, I will create two smaller datasets: CHN and CHS. 

I can do this in pyRAD, by rerunning s67. 



##EMMAX test

I will try to run EMMAX, using elevation i.s.o temp for the first test: 

Tutorial: 

http://genetics.cs.ucla.edu/emmax/install.html

And Victoria Sork also sent a tutorial 

Create the plink files and transfer to the mac: 

```
plink --file CH436.final --recode12 --output-missing-genotype 0 --transpose --out CH436emmax


scp -r alexjvr@gdcsrv1.ethz.ch:/gdc_home4/alexjvr/CH530SNPfiltering/filteredforPCA/CH436emmax* .
```

And create a phenotype file:

famID, indivID, phenotype (elevation in my case)

Make sure everything is in the same folder

Generate an IBS kinship matrix in emmax
```
/Users/alexjvr/Applications/emmax-beta-07Mar2010/emmax-kin -v -h -s -d 10 CH436emmax
```



Copy over to the mac: 
And copy the .raw file over to the mac
```
scp -r alexjvr@gdcsrv1.ethz.ch:/gdc_home4/alexjvr/CH530SNPfiltering/filteredforPCA/CH436.final.plink.raw .
```








##Data conversion

To read data into R, it should be in the correct format. VCFtools can convert into various formats: 

For the subset data (input for the second command is the .ped file from the first conversion. The --recodeA command changes the SNPs to binary, inferring one to be 0 and the alt allele a different number. 
```
vcftools --vcf subset.Final.vcf --out subset.Final.plink --plink
plink --file subsetFinal.plink --out subsetFinalraw.plink --recodeA
```




Output for my data
```
** For gPLINK compatibility, do not use '.' in --out **
16331 (of 16331) markers to be included from [ subsetFinal.plink.map ]
Warning, found 137 individuals with ambiguous sex codes
Writing list of these individuals to [ subsetFinalraw.plink.nosex ]
137 individuals read from [ subsetFinal.plink.ped ] 
0 individuals with nonmissing phenotypes
Assuming a disease phenotype (1=unaff, 2=aff, 0=miss)
Missing phenotype value is also -9
0 cases, 0 controls and 137 missing
0 males, 0 females, and 137 of unspecified sex
Before frequency and genotyping pruning, there are 16331 SNPs
137 founders and 0 non-founders found
Total genotyping rate in remaining individuals is 0.746582
0 SNPs failed missingness test ( GENO > 1 )
0 SNPs failed frequency test ( MAF < 0 )
After frequency and genotyping pruning, there are 16331 SNPs
After filtering, 0 cases, 0 controls and 137 missing
After filtering, 0 males, 0 females, and 137 of unspecified sex
Writing recoded file to [ subsetFinalraw.plink.raw ] 
```

So, 16331 SNPs in total. 137 individuals. 

And copy the .raw file over to the mac
```
scp -r alexjvr@gdcsrv1.ethz.ch:/gdc_home4/alexjvr/ddRADpipeline/pyrad/SNPfiltering/subsetFinalraw.plink.raw .
```

Read this file into R. This reads the PLINK generated .raw data file into R, and converts it into a genlight object (Class S4)
```
setwd("~/phd_20150212/Analysis/ddRAD")
read.PLINK("subsetFinalraw.plink.raw")
```

###Christine G. recommends redoing analyses using different filters to see how sensitive they are. So change 

- the genotyping rate
            Don't use 100% genotyping rate. This is likely to include paralogues, particularly with low coverage data

- allowed maf
            Christine recommends choosing a subset of the data where the maf within each population is not too low. She filtered for 5%. This is because between species Fst is likely to be inflated if such markers are used. This is probably not true within species. 


##Tassle



##SNPRelate



##Adegenet

This is a package for population genetics in R. 

More info and tutorials here: http://adegenet.r-forge.r-project.org/

Convert vcf file to genepop format in PGDspider (http://www.cmpg.unibe.ch/software/PGDSpider/). 
 NB!! The genpop format doesn't have a header, so won't work properly. Add a file name in the first line of the file. 
 
 Pop information can be imported during file conversion with pgdspider. 
 
 






####Random code


R code for checking whether Haplotype diversity (or other measure) is corrolated wiht latitude or altitude
```
CH <- Table1_elev_CH_20150727

plot(CH$Long., CH$Haplotype.diversity..Hd., main="Elevation ~ haplotype diversity")

```

The input I used was a modification of Table1 from the mtDNA dataset
![alt_txt][data]
[data]:https://cloud.githubusercontent.com/assets/12142475/8954946/3819df28-35e9-11e5-8e6a-ae9fe3fe1941.png


In this case there is no correlation

![alt_txt][input]
[input]:https://cloud.githubusercontent.com/assets/12142475/8954956/556fffd0-35e9-11e5-8c5e-c32f458597ad.png





##Basic stats that I'm working on 

```
##Basic stats

##PCA
library("ade4")
library("adegenet")
library("pegas")


CH.plink <- read.PLINK("CH436.plink.raw") ##reads a plink file in as genlight object

CH.plink #check that it is a genlight object

indNames(CH.plink)

CH436_pop.names <- read.table("~/phd_20150212/Analysis/ddRAD/CH530ddRAD/CH436_pop.names.csv", header=T, quote="\"") #read in the population names file
CH436_pop.names.factors <- as.factor(CH436_pop.names$PopID) #and convert to a factor
summary(CH436_pop.names)

pop(CH.plink) <- (CH436_pop.names.factors) #assign population names from a text file
pop(CH.plink)  ##and check that they are correct

temp <- table(unlist(other(CH.plink)))
barplot(temp,main="distribution of NoAlleles per locus",xlab="Number of Alleles",ylab="Number of sites",col=heat.colors(4))

myFreq <- glMean(CH.plink)
hist(myFreq, proba=T, col="gold", xlab="Allele Frequencies", main="Distribution of (2nd) allele frequencies")
temp <- density(myFreq)
lines(temp$x, temp$y,*1.8, lwd=3)

##Pop structure
pca1 <- glPca(CH.plink) ##This displays a barplot of the eigenvalues and asks the user for a number of retained principal components

scatter(pca1, posi="bottomright") #scatter plot of PC 1& 2
title("PCA of CH Rana temporaria, axes 1 and 2")

#NJ tree
library(ape)
tre <- nj(dist(as.matrix(CH.plink)))
tre


```


