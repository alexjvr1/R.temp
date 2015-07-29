#Basic Statistics

For an initial look at the data, summary stats, and population genetics. 

I will mainly use adegenet and resources specified on their website: 

http://adegenet.r-forge.r-project.org/


Working through the following tutorial: Analysing Genome wide SNPs using AdeGenet 
http://adegenet.r-forge.r-project.org/files/tutorial-genomics.pdf

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
