#SE ddRAD data

13 Oct 2015

I received the sequencing data back to today for the final sequencing lanes. 

I will run FastQC & demultiplex samples

Then I will start a pyRAD run with the aim to have data to present at the end of the month. 

Step 1 & 2 were conducted on fgcz server: 

I demultiplexed all samples using process_radtags (generic code)

```
/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150703_RUN199/20150703.A-H10_R1.fastq.gz  -o ./demultiplexed/H10 -y fastq -b ./barcodes/H10_barcodes --disable_rad_check -r -D
```

and then trimmed using trimmomatic

```
screen -S TrimSubset -L
for i in *.fq; do  java -jar /usr/local/ngseq/src/Trimmomatic-0.33/trimmomatic-0.33.jar SE $i $i.trim ILLUMINACLIP:/usr/local/ngseq/src/Trimmomatic-0.33/adapters/TruSeq3-SE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36;done
```


Then I split the dataset into 2: 99 to gdcsrv1 and 94 to fgcz. And started pyRAD s2345

on FGCZ: 


```

/usr/local/ngseq/src/pyrad-3.0.61/pyrad/pyRAD.py -n





==** parameter inputs for pyRAD version 3.0.61  **======================== affected step ==
./                        ## 1. Working directory                                 (all)
              ## 2. Loc. of non-demultiplexed files (if not line 18)  (s1)
              ## 3. Loc. of barcode file (if not line 18)             (s1)
/usr/local/ngseq/src/vsearch-1.6.0-linux-x86_64/bin/vsearch                   ## 4. command ($
/usr/local/ngseq/stow/muscle-3.8.31/bin/muscle                    ## 5. command (or path) to $
TGCAG                     ## 6. Restriction overhang (e.g., C|TGCAG -> TGCAG)     (s1,s2)
12                         ## 7. N processors (parallel)                           (all)
6                         ## 8. Mindepth: min coverage for a cluster              (s4,s5)
4                         ## 9. NQual: max # sites with qual < 20 (or see line 20)(s2)
.94                       ## 10. Wclust: clustering threshold as a decimal        (s3,s6)
ddrad                       ## 11. Datatype: rad,gbs,pairgbs,pairddrad,(others:see docs)(all)
10                         ## 12. MinCov: min samples in a final locus             (s7)
100                         ## 13. MaxSH: max inds with shared hetero site          (s7)
SEfgczC94                 ## 14. Prefix name for final output (no spaces)         (s7)
==== optional params below this line ===================================  affected step ==
                       ## 15.opt.: select subset (prefix* only selector)            (s2-s7)
                       ## 16.opt.: add-on (outgroup) taxa (list or prefix*)         (s6,s7)
                       ## 17.opt.: exclude taxa (list or prefix*)                   (s7)
./SEfgczPyRAD/                      ## 18.opt.: loc. of de-multiplexed data                  $
                       ## 19.opt.: maxM: N mismatches in barcodes (def= 1)          (s1)
                       ## 20.opt.: phred Qscore offset (def= 33)                    (s2)
                       ## 21.opt.: filter: def=0=NQual 1=NQual+adapters. 2=strict   (s2)
                       ## 22.opt.: a priori E,H (def= 0.001,0.01, if not estimated) (s5)
                       ## 23.opt.: maxN: max Ns in a cons seq (def=5)               (s5)
                       ## 24.opt.: maxH: max heterozyg. sites in cons seq (def=5)   (s5)
                       ## 25.opt.: ploidy: max alleles in cons seq (def=2;see docs) (s4,s5)
                       ## 26.opt.: maxSNPs: (def=100). Paired (def=100,100)         (s7)
                       ## 27.opt.: maxIndels: within-clust,across-clust (def. 3,99) (s3,s7)
                       ## 28.opt.: random number seed (def. 112233)              (s3,s6,s7)
                       ## 29.opt.: trim overhang left,right on final loci, def(0,0) (s7)
              *         ## 30.opt.: output formats: p,n,a,s,v,u,t,m,k,g,* (see docs) (s7)
                       ## 31.opt.: maj. base call at depth>x<mindepth (def.x=mindepth) (s5)
               50        ## 32.opt.: keep trimmed reads (def=0). Enter min length.    (s2)
                       ## 33.opt.: max stack size (int), def= max(500,mean+2*SD)    (s3)
                       ## 34.opt.: minDerep: exclude dereps with <= N copies, def=1 (s3)
                       ## 35.opt.: use hierarchical clustering (def.=0, 1=yes)      (s6)
                       ## 36.opt.: repeat masking (def.=1='dust' method, 0=no)      (s3,s6)



screen -S SEpyradfgcz
/usr/local/ngseq/src/pyrad-3.0.61/pyrad/pyRAD.py -p params.txt -s 2345
```

and on the GDCsrv1

```
module load pyRAD3.0.6
pyrad -p params.txt -s 234567
```
Running both on GDC now, since I can't run enough cores on fgcz. 


All SNPs called on 21 Oct 2015 (8 days for 193 samples across 2 clusters)

Following the ddRAD_pipeline protocol to filter the Bombina VCF file: 

First keep only SNPs that have been genotyped in 50% of individuals, with MAC of 3

```
vcftools --vcf SEpyradGDCc94.vcf --max-missing 0.5 --mac 3 --recode --recode-INFO-all --out SE193mac3

VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf SEpyradGDCc94.vcf
	--recode-INFO-all
	--mac 3
	--max-missing 0.5
	--out SE193mac3
	--recode

Eighth Header entry should be INFO: INFO    
After filtering, kept 193 out of 193 Individuals
Outputting VCF file...
After filtering, kept 46506 out of a possible 807731 Sites
Run Time = 39.00 seconds
```
Keepign 46500 loci

Look at the per individual missingness
```
vcftools --vcf SE193mac3.recode.vcf --missing-indv

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


![alt txt][imissSE]
[imissSE]:https://cloud.githubusercontent.com/assets/12142475/10632183/f55d7a96-77e2-11e5-8234-28759fd4bc0f.png


So most of the missingness seems to be <50%. But I need to check which individuals will drop out (and if whole populations are dropping out). 

For a 50% missingness threshold

```
mawk ‘$5 > 0.5’ out.imiss | cut -f1 > lowDP.indiv

wc -l lowDP.indiv
```

So this drop 57 individuals. 

Let's see who they are:
```
cat lowDP.indiv

F20.fq.trim
F24.fq.trim
F25.fq.trim
F28.fq.trim
F30.fq.trim
Gra02.fq.trim
Ho_03.fq.trim
Ho_04.fq.trim
Ho_05.fq.trim
Ho_06.fq.trim
Ho_07.fq.trim
Ho_09.fq.trim
Ho_10.fq.trim
Ho_11.fq.trim
Ho_13.fq.trim
Ho_14.fq.trim
Ho_15.fq.trim
Ho_16.fq.trim
Ho_17.fq.trim
Ho_18.fq.trim
LT1_10.fq.trim
LT2_01.fq.trim
LT2_02.fq.trim
LT2_03.fq.trim
LT2_05.fq.trim
LT2_07.fq.trim
LT2_08.fq.trim
LT2_09.fq.trim
LT3_03.fq.trim
LT3_05.fq.trim
LT3_08.fq.trim
SL_01.fq.trim
SL_02.fq.trim
SL_03.fq.trim
SL_04.fq.trim
SL_05.fq.trim
SL_06.fq.trim
SL_07.fq.trim
SL_08.fq.trim
SL_09.fq.trim
SL_10.fq.trim
SL_11.fq.trim
SL_12.fq.trim
SL_13.fq.trim
SL_14.fq.trim
SL_15.fq.trim
SL_16.fq.trim
SL_17.fq.trim
SL_18.fq.trim
SL_19.fq.trim
Um_Gr03.fq.trim
Um_UT3_06.fq.trim
Um_UT3_09.fq.trim
Um_UT3_20.fq.trim
Upp_O07.fq.trim
```

Populations that will drop out: 

Um_UT3 (3/5)  - Umea

SL (19/19)  - Skane

LT2 (7/10)  - Lulea
 
Hö (14/20) - Skane

If I change the cut-off to 0.7 or 0.8, less individuals are filtered. It seems like SL is the worst population. This seems to be because H23 hiseq lane was poorly sequenced. I have a rerun of this data. So I should concatenate the samples for this and rerun pyrad. 

SL is a population from Skone. I need to check the Hö run as well: they're also mostly from H23. 

If this is sorted out, then the dataset should be complete. 

In the meantime, I will continue to look at the populations after filtering for 0.5 missingness: 

```
vcftools --vcf SE193mac3.recode.vcf --remove lowDP.indiv --recode --recode-INFO-all --out SE.imiss50

VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf SE193mac3.recode.vcf
	--exclude lowDP.indiv
	--recode-INFO-all
	--out SE.imiss50
	--recode

Excluding individuals in 'exclude' list
After filtering, kept 138 out of 193 Individuals
Outputting VCF file...
After filtering, kept 46506 out of a possible 46506 Sites
Run Time = 6.00 seconds
```
Convert to plink format & transfer everything to the mac. 

```
vcftools --vcf SE.imiss50.recode.vcf --out SE138.final --plink
plink --file SE138.final --out SE138.final.plink --recodeA


+++ PLINK 1.9 is now available! See above website for details +++ 

Writing this text to log file [ SE138.final.plink.log ]
Analysis started: Wed Oct 21 12:03:43 2015

Options in effect:
	--file SE138.final
	--out SE138.final.plink
	--recodeA

** For gPLINK compatibility, do not use '.' in --out **
45741 (of 45741) markers to be included from [ SE138.final.map ]
Warning, found 138 individuals with ambiguous sex codes
Writing list of these individuals to [ SE138.final.plink.nosex ]
138 individuals read from [ SE138.final.ped ] 
0 individuals with nonmissing phenotypes
Assuming a disease phenotype (1=unaff, 2=aff, 0=miss)
Missing phenotype value is also -9
0 cases, 0 controls and 138 missing
0 males, 0 females, and 138 of unspecified sex
Before frequency and genotyping pruning, there are 45741 SNPs
138 founders and 0 non-founders found
Total genotyping rate in remaining individuals is 0.789138
0 SNPs failed missingness test ( GENO > 1 )
0 SNPs failed frequency test ( MAF < 0 )
After frequency and genotyping pruning, there are 45741 SNPs
After filtering, 0 cases, 0 controls and 138 missing
After filtering, 0 males, 0 females, and 138 of unspecified sex
Writing recoded file to [ SE138.final.plink.raw ] 

Analysis finished: Wed Oct 21 12:03:47 2015
```


Type this in terminal on the computer where the data will be transferred to
```
scp -r alexjvr@gdcsrv1.ethz.ch:SE193filtering/* .
```

make a file with all the population names. This can be obtained with mawk: 

```
mawk ‘$5 < 0.5’ out.imiss | cut -f1 > final.indiv

wc -l final.indiv

cat final.indiv
```

This is a hack. I have to figure out how to actually get a list of sample names from these files... 

But now I can use this to generate a population file. 


##Fst

Check for global Fst using vcftools

1. make files containing all the individual names for all the populations

DE_B.pop  DE_W.pop  Gra.pop  KirG.pop  LT1.pop  LT3.pop  Um.Gr.pop   Upp_K.pop
DE_K.pop  FIN.pop   Ho.pop   KirL.pop  LT2.pop  SF.pop   Um.Taf.pop  Upp_O.pop

So, I will check for a global Fst, and then within each region: 


Germany: DE_B, K, W 
```
--vcf SE.imiss50.recode.vcf --weir-fst-pop DE_B.pop --weir-fst-pop DE_K.pop --weir-fst-pop DE_W.pop --out DE.fst

After filtering, kept 138 out of 138 Individuals
Outputting Weir and Cockerham Fst estimates.
Weir and Cockerham mean Fst estimate: 0.02488
Weir and Cockerham weighted Fst estimate: 0.044847
After filtering, kept 46506 out of a possible 46506 Sites
```


Kiruna: KirG, L 
```
vcftools --vcf SE.imiss50.recode.vcf --weir-fst-pop KirL.pop --weir-fst-pop KirG.pop --out Kir.fst

Weir and Cockerham mean Fst estimate: 0.044349
Weir and Cockerham weighted Fst estimate: 0.072487
```



Kir & Fin: KirG, L, FIN
```
--vcf SE.imiss50.recode.vcf --weir-fst-pop KirL.pop --weir-fst-pop KirG.pop --weir-fst-pop FIN.pop --out Kir.FIN.fst

Weir and Cockerham mean Fst estimate: 0.088804
Weir and Cockerham weighted Fst estimate: 0.11858
```



Lulea: LT1,2,3
```
--vcf SE.imiss50.recode.vcf --weir-fst-pop LT1.pop --weir-fst-pop LT2.pop --weir-fst-pop LT3.pop --out Lulea.fst

Weir and Cockerham mean Fst estimate: 0.054745
Weir and Cockerham weighted Fst estimate: 0.073497
```



Uppsala: UppK, O, Gra -
```
vcftools --vcf SE.imiss50.recode.vcf --weir-fst-pop Upp_K.pop --weir-fst-pop Upp_O.pop --weir-fst-pop Gra.pop --out Uppsala.fst

Weir and Cockerham mean Fst estimate: 0.083389
Weir and Cockerham weighted Fst estimate: 0.11299
```


Umea: Um.Gr, Taf  -
```
 vcftools --vcf SE.imiss50.recode.vcf --weir-fst-pop Um.Gr.pop --weir-fst-pop Um.Taf.pop --out Umea.fst

Weir and Cockerham mean Fst estimate: 0.021277
Weir and Cockerham weighted Fst estimate: 0.039893
```

Skane: SF, Ho
```
vcftools --vcf SE.imiss50.recode.vcf --weir-fst-pop

Weir and Cockerham mean Fst estimate: 0.0088461
Weir and Cockerham weighted Fst estimate: 0.025573
```


And overall for SE
```
vcftools --vcf SE.imiss50.recode.vcf --weir-fst-pop Upp_K.pop --weir-fst-pop Upp_O.pop --weir-fst-pop Gra.pop --weir-fst-pop LT1.pop --weir-fst-pop LT2.pop --weir-fst-pop LT3.pop --weir-fst-pop KirL.pop --weir-fst-pop KirG.pop --weir-fst-pop FIN.pop --weir-fst-pop DE_B.pop --weir-fst-pop DE_K.pop --weir-fst-pop DE_W.pop --out SE.fst


Weir and Cockerham mean Fst estimate: 0.29046
Weir and Cockerham weighted Fst estimate: 0.4069
```






And the same thing after filtering the vcf for MAF 0.25
```
vcftools --vcf subset.final.vcf --maf 0.25 --recode --recode-INFO-all --out subset.final.maf25

Parameters as interpreted:
	--vcf SE.imiss50.recode.vcf
	--recode-INFO-all
	--maf 0.25
	--out SE.final.maf0.25
	--recode

After filtering, kept 138 out of 138 Individuals
Outputting VCF file...
After filtering, kept 8182 out of a possible 46506 Sites
Run Time = 3.00 seconds
```

So now we're down to only 8000 SNPs. If I rerun the Fst data: 
Germany: DE_B, K, W 
```
vcftools --vcf SE.final.maf0.25.recode.vcf --weir-fst-pop DE_B.pop --weir-fst-pop DE_K.pop --weir-fst-pop DE_W.pop --out DE.fst

After filtering, kept 138 out of 138 Individuals
Outputting Weir and Cockerham Fst estimates.
Weir and Cockerham mean Fst estimate: 0.02488
Weir and Cockerham weighted Fst estimate: 0.044847
After filtering, kept 46506 out of a possible 46506 Sites

MAF0.25
Weir and Cockerham mean Fst estimate: 0.021994
Weir and Cockerham weighted Fst estimate: 0.043397
```


Kiruna: KirG, L 
```
vcftools --vcf SE.final.maf0.25.recode.vcf --weir-fst-pop KirL.pop --weir-fst-pop KirG.pop --out Kir.fst

Weir and Cockerham mean Fst estimate: 0.044349
Weir and Cockerham weighted Fst estimate: 0.072487

MAF0.25
Weir and Cockerham mean Fst estimate: 0.050283
Weir and Cockerham weighted Fst estimate: 0.073653
```



Kir & Fin: KirG, L, FIN
```
vcftools --vcf SE.final.maf0.25.recode.vcf --weir-fst-pop KirL.pop --weir-fst-pop KirG.pop --weir-fst-pop FIN.pop --out Kir.FIN.fst

Weir and Cockerham mean Fst estimate: 0.088804
Weir and Cockerham weighted Fst estimate: 0.11858

MAF0.25
Weir and Cockerham mean Fst estimate: 0.097586
Weir and Cockerham weighted Fst estimate: 0.11947
```



Lulea: LT1,2,3
```
vcftools --vcf SE.final.maf0.25.recode.vcf --weir-fst-pop LT1.pop --weir-fst-pop LT2.pop --weir-fst-pop LT3.pop --out Lulea.fst

Weir and Cockerham mean Fst estimate: 0.054745
Weir and Cockerham weighted Fst estimate: 0.073497

MAF0.25
Weir and Cockerham mean Fst estimate: 0.062096
Weir and Cockerham weighted Fst estimate: 0.075543
```



Uppsala: UppK, O, Gra -
```
vcftools --vcf SE.final.maf0.25.recode.vcf --weir-fst-pop Upp_K.pop --weir-fst-pop Upp_O.pop --weir-fst-pop Gra.pop --out Uppsala.fst

Weir and Cockerham mean Fst estimate: 0.083389
Weir and Cockerham weighted Fst estimate: 0.11299

MAF0.25
Weir and Cockerham mean Fst estimate: 0.099522
Weir and Cockerham weighted Fst estimate: 0.11764
```


Umea: Um.Gr, Taf  -
```
vcftools --vcf SE.final.maf0.25.recode.vcf --weir-fst-pop Um.Gr.pop --weir-fst-pop Um.Taf.pop --out Umea.fst

Weir and Cockerham mean Fst estimate: 0.021277
Weir and Cockerham weighted Fst estimate: 0.039893

MAF0.25
Weir and Cockerham mean Fst estimate: 0.027194
Weir and Cockerham weighted Fst estimate: 0.043908
```

Skane: SF, Ho
```
vcftools --vcf SE.final.maf0.25.recode.vcf --weir-fst-pop SF.pop --weir-fst-pop Ho.pop --out Skane.fst

Weir and Cockerham mean Fst estimate: 0.0088461
Weir and Cockerham weighted Fst estimate: 0.025573

MAF0.25
Weir and Cockerham mean Fst estimate: 0.0057413
Weir and Cockerham weighted Fst estimate: 0.024687
```


And overall for SE
```
vcftools --vcf SE.final.maf0.25.recode.vcf --weir-fst-pop Upp_K.pop --weir-fst-pop Upp_O.pop --weir-fst-pop Gra.pop --weir-fst-pop LT1.pop --weir-fst-pop LT2.pop --weir-fst-pop LT3.pop --weir-fst-pop KirL.pop --weir-fst-pop KirG.pop --weir-fst-pop FIN.pop --weir-fst-pop DE_B.pop --weir-fst-pop DE_K.pop --weir-fst-pop DE_W.pop --out SE.fst


Weir and Cockerham mean Fst estimate: 0.29046
Weir and Cockerham weighted Fst estimate: 0.4069

Weir and Cockerham mean Fst estimate: 0.42109
Weir and Cockerham weighted Fst estimate: 0.42914

```

##colours for unrooted dendrogram in Adobe illustrator

Yellow: FFCC66

Green: 009933

Blue: 0099CC

Light Blue: 99CCFF



###SessionII

1. Arrange all samples and data
2. concatenate samples as needed
3. Choose subset with which to test pyRAD clustering threshold
4. Run pyRAD tests clustering 90-98%


There are 193 Swedish samples from 19 populations, 7 latitudes. Samples for which there are two runs: 

Hö 01 (H23 + H25)

Hö 08 (H22 + H25)

Hö 12 (H23 - lost in cleanup + H20)
Hö 19 (H23 + H25)

Hö 20 (H23 + H25)

And the following two plates had 2 runs each: 

H22 & H23

So: SL & Hö are affected. 

Country|Population| n |lat | long
:---:|:---:|:--:|:--:|:--:
Germany|B|10|52.4647|10.0844
Germany|K| 10 |52.55825	|9.362521
Germany|W|  10|	52.589665|	10.144619
Skåne |SF|10	|55.55757|	13.63819
Skåne|skåne1 (Hö)|20	|55.722889	|13.286917
Skåne|skåne3 (SL)|19|	55.8585|	13.763833
Uppsala |Gra|10|	59.877851|	17.667242
Uppsala|Ko|10|	59.890968|	17.242389
Uppsala|O|10|	60.178358|	17.854461
Umeå |Gro|10|	63.791632|	20.367129
Umeå|UT3|4|	63.658281|	20.29822
Umeå|Taf|10|	63.830139|	20.486056
Luleå|LT1|10|	65.684418|	22.212979
Luleå|LT2|10|	65.749836|	21.601634
Luleå|LT3|10|	65.582783|	22.31932
Gällivare/Kiruna|G|10|	67.111307|	20.656016
Gällivare/Kiruna|L1|10|	67.051924|	21.22396
Finland North|F	|10|69.04445|	20.804544



Samples in the cluster optimisation: 

Pop| Samples
:--:|:--:
DE_B |01 + 02
DE_K| 01 + 02
DE_W | 01 + 02
Skåne_Hö |01 + 02
Skåne_SF|01 + 02
Skåne_SL| 01 + 02
Uppsala_Gra |03 + 04
Uppsala_K|01 + 02
Uppsala_O|01 + 04
Um_Gr| 01 + 02
Um_Taf| 01 + 02
Luleå_LT1| 01 + 02
Luleå_LT2| 04 + 06
Luleå_LT3| 01 + 02
Kir_G| 01 + 02
Kir_L | 01 + 02
FIN | 21-23, 29

**Samples were chosen as those that had not dropped out during the test run. (Most of the dropout was likely because I didn't concatenate the repeated lanes/samples)

**For SL, samples were chosen at random, since non-concatenated samples were used for this whole population during the test run. 



##PyradOpt

I will run 2 analyses: 

1. All 19 populations, 36 indivs

2. Excluding 3 DE populations: 18 pops, 30 indivs


I will test clustering 90-99%


On GDCsrv 1 at 16:40

```
module load pyRAD3.0.6

pyrad -n



==** parameter inputs for pyRAD version 3.0.6  **======================== affected step ==
./                        ## 1. Working directory                                 (all)
              ## 2. Loc. of non-demultiplexed files (if not line 16)  (s1)
              ## 3. Loc. of barcode file (if not line 16)             (s1)
vsearch                   ## 4. command (or path) to call vsearch (or usearch)    (s3,s6)
muscle                    ## 5. command (or path) to call muscle                  (s3,s7)
TGCAG                     ## 6. Restriction overhang (e.g., C|TGCAG -> TGCAG)     (s1,s2)
20                         ## 7. N processors (parallel)                           (all)
6                         ## 8. Mindepth: min coverage for a cluster              (s4,s5)
4                         ## 9. NQual: max # sites with qual < 20 (or see line 20)(s2)
.90                       ## 10. Wclust: clustering threshold as a decimal        (s3,s6)
ddrad                       ## 11. Datatype: rad,gbs,pairgbs,pairddrad,(others:see docs)(all)
4                         ## 12. MinCov: min samples in a final locus             (s7)
40                         ## 13. MaxSH: max inds with shared hetero site          (s7)
SEc90d6m4p3                 ## 14. Prefix name for final output (no spaces)         (s7)
==== optional params below this line ===================================  affected step ==
                       ## 15.opt.: select subset (prefix* only selector)            (s2-s7)
                       ## 16.opt.: add-on (outgroup) taxa (list or prefix*)         (s6,s7)
                       ## 17.opt.: exclude taxa (list or prefix*)                   (s7)
  /gdc_home4/alexjvr/SEFinalSamples/SEpyradOpt/samples/                     ## 18.opt.: loc. of de-multiplexed data                      (s2)
                       ## 19.opt.: maxM: N mismatches in barcodes (def= 1)          (s1)
                       ## 20.opt.: phred Qscore offset (def= 33)                    (s2)
                       ## 21.opt.: filter: def=0=NQual 1=NQual+adapters. 2=strict   (s2)
                       ## 22.opt.: a priori E,H (def= 0.001,0.01, if not estimated) (s5)
                       ## 23.opt.: maxN: max Ns in a cons seq (def=5)               (s5)
                       ## 24.opt.: maxH: max heterozyg. sites in cons seq (def=5)   (s5)
                       ## 25.opt.: ploidy: max alleles in cons seq (def=2;see docs) (s4,s5)
                       ## 26.opt.: maxSNPs: (def=100). Paired (def=100,100)         (s7)
                       ## 27.opt.: maxIndels: within-clust,across-clust (def. 3,99) (s3,s7)
                       ## 28.opt.: random number seed (def. 112233)              (s3,s6,s7)
                       ## 29.opt.: trim overhang left,right on final loci, def(0,0) (s7)
          *             ## 30.opt.: output formats: p,n,a,s,v,u,t,m,k,g,* (see docs) (s7)
                       ## 31.opt.: maj. base call at depth>x<mindepth (def.x=mindepth) (s5)
           50            ## 32.opt.: keep trimmed reads (def=0). Enter min length.    (s2)
                       ## 33.opt.: max stack size (int), def= max(500,mean+2*SD)    (s3)
                       ## 34.opt.: minDerep: exclude dereps with <= N copies, def=1 (s3)
                       ## 35.opt.: use hierarchical clustering (def.=0, 1=yes)      (s6)
                       ## 36.opt.: repeat masking (def.=1='dust' method, 0=no)      (s3,s6)
                       ## 37.opt.: vsearch max threads per job (def.=6; see docs)   (s3,s6)
==== optional: list group/clade assignments below this line (see docs) ==================


pyrad -p params.txt -s 234567
```

on GDCsrv 2 at 17:00

Keeping track of memory usage during the different steps

Step 2||Step 3||Step 4|| Step 5 || Step 6||
:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
VIRT|RES|VIRT|RES|VIRT|RES|VIRT|RES|VIRT|RES
425M*x|25M*x|3500-5000M*x|1.2-1.5G*x|2500M|2.1G|350M|30M|2500M|2.1G



##excluding DE populations  19Nov

I can use the runs from the full dataset, and run s67 with DE samples excluded. The exclusion option is only for s7. But I need to exclude samples at s6 (across sample clustering). 

Set up new pyrad run by moving edits/ and clustxx/ to 9xSE folders. The DE samples will be kept in the folders in the origial run (i.e 91/, 92/, etc.). These data can be moved back to the original folders afterwards. 

- I deleted the DE samples by mistake, so I will need to rerun these during the final SE run.

- I may have to rerun these samples for .98 clustering as this had not completed writing the output files before I moved the /edits and /clust.98 folders: 


Start the first SE only pyRAD (.90 clustering) on gdcsrv2. 19 NOV 10:45. 

All completed on 20 Nov ~11:00. 

##Optimisation: plots

To check what the optimal clustering threshold is, I plotted the following parameters. 

1. He vs cluster  (stats/Pi_*)

2. Error vs cluster (stats/Pi_*)

3. nloci vs cluster (stats/SExxx)

4. % polymorphism (stats/s5_*)

5. n filtered paralogs (stats/s5* -> f2loci-f1loci)


I export everything to an excel file (SE_pyRADopt_20151120)

Import into R, and run the following code: 

```
##PyRAD opt

#import data sheet 1: HEvsclustering


library(ggplot2)
install.packages("reshape2")
library(reshape2)

rm(list=ls())       #clear the workspace
library(reshape2)    #package for reshaping the data
library(ggplot2)
 
## Het & error
SE_allHE <- read.csv("SE_allHE_20151120.csv")

head(SE_allHE)


data<-melt(SE_allHE[,3:12])  #3:10 selects just the columns that we're interested in (nloci_90 to nloci_99)

head(data)
figure<-ggplot(data,aes(x=variable,y=value))+geom_boxplot()
figure

##nloci vs cluster SEall

SE_allnloci <- read.csv("SE_nloci_20151120.csv")

head(SE_allnloci)

data<-melt(SE_allnloci[,3:12])  #3:10 selects just the columns that we're interested in (nloci_90 to nloci_99)

head(data)
figure<-ggplot(data,aes(x=variable,y=value))+geom_boxplot()
figure

##npoly vs cluster

SE_allnpoly <- read.csv("SE_npoly_20151120.csv")

head(SE_allnpoly)

data<-melt(SE_allnpoly[,63:72])  #3:10 selects just the columns that we're interested in (nloci_90 to nloci_99)

head(data)
figure<-ggplot(data,aes(x=variable,y=value))+geom_boxplot()
figure

##paralogs vs cluster SEall

SE_allnpoly <- read.csv("SE_npoly_20151120.csv")

head(SE_allnpoly)

data<-melt(SE_allnpoly[,33:42])  #3:10 selects just the columns that we're interested in (nloci_90 to nloci_99)

head(data)
figure<-ggplot(data,aes(x=variable,y=value))+geom_boxplot()
figure

## Het & error SE only
SE_allHE <- read.csv("SEonly_HE_20151120.csv")

head(SE_allHE)
#order the data by the header: df[,oder(names(df))]

data<-melt(SE_allHE[,3:12])  #3:10 selects just the columns that we're interested in (nloci_90 to nloci_99)

head(data)
figure<-ggplot(data,aes(x=variable,y=value))+geom_boxplot()
figure

##nloci vs cluster SEonly

SE_allnloci <- read.csv("SEonly_nloci_20151120.csv")

head(SE_allnloci)

data<-melt(SE_allnloci[,3:11])  #3:10 selects just the columns that we're interested in (nloci_90 to nloci_99)

head(data)
figure<-ggplot(data,aes(x=variable,y=value))+geom_boxplot()
figure

##npoly vs cluster SEonly

SE_allnpoly1 <- read.csv("SEonly_paralogs_20151120.csv")

head(SE_allnpoly1)

SE_allnpoly <- SE_allnpoly1[,order(names(SE_allnpoly1))] #order the data by the header

head(SE_allnpoly)


data<-melt(SE_allnpoly[,61:70])  #3:10 selects just the columns that we're interested in (nloci_90 to nloci_99)

head(data)

figure<-ggplot(data,aes(x=variable,y=value))+geom_boxplot()
figure

##paralogs vs cluster SEonly

SE_allnpoly1 <- read.csv("SEonly_paralogs_20151120.csv")

head(SE_allnpoly1)

SE_allnpoly <- SE_allnpoly1[,order(names(SE_allnpoly1))] #order the data by the header

head(SE_allnpoly)


data<-melt(SE_allnpoly[,51:60])  #3:10 selects just the columns that we're interested in (nloci_90 to nloci_99)

head(data)

figure<-ggplot(data,aes(x=variable,y=value))+geom_boxplot()
figure





```

##SE all ponds

1. Het vs clusters

![alt txt][SEall.1]
[SEall.1]:https://cloud.githubusercontent.com/assets/12142475/11308878/fcb30d2e-8fc0-11e5-8e0f-fc9737970033.png

2. Error vs clusters

![alt txt][SEall.2]
[SEall.2]:https://cloud.githubusercontent.com/assets/12142475/11308881/fcb8c458-8fc0-11e5-9f3a-d81780a1e816.png

3. nloci vs clusters

![alt txt][SEall.3]
[SEall.3]:https://cloud.githubusercontent.com/assets/12142475/11308876/fcb1d490-8fc0-11e5-84c1-dc1fde654313.png

4. % polymorphism vs clusters

![alt txt][SEall.4]
[SEall.4]:https://cloud.githubusercontent.com/assets/12142475/11308877/fcb2f488-8fc0-11e5-862a-d6066f5fefbf.png

5. n filtered paralogs vs clusters 
![alt txt][SEall.5]
[SEall.5]:https://cloud.githubusercontent.com/assets/12142475/11308879/fcb3ad2e-8fc0-11e5-8d5e-2ae73b087a1d.png

##only SE ponds (no DE)

1. Het vs clusters

![alt txt][SE.1]
[SE.1]:https://cloud.githubusercontent.com/assets/12142475/11308880/fcb584aa-8fc0-11e5-86b5-0356ced3509f.png

2. Error vs clusters

![alt txt][SE.2]
[SE.2]:https://cloud.githubusercontent.com/assets/12142475/11308884/fccadd78-8fc0-11e5-8eaf-9cec41290b86.png

3. nloci vs clusters

![alt txt][SE.3]
[SE.3]:https://cloud.githubusercontent.com/assets/12142475/11308883/fcca8b48-8fc0-11e5-9a20-9e87d3ef1f56.png

4. % polymorphism vs clusters

![alt txt][SE.4]
[SE.4]:https://cloud.githubusercontent.com/assets/12142475/11308882/fcc7d920-8fc0-11e5-8397-4c5ac806a5cd.png

5. n paralogs removed vs clusters

![alt txt][SE.5]
[SE.5]:https://cloud.githubusercontent.com/assets/12142475/11308885/fccada08-8fc0-11e5-9541-6e19bfc06a8f.png


Interpretation: 

1. No real difference between SE and SE + DE datasets

2. 0.94 or 0.95 looks like the optimal clustering threshold. 









