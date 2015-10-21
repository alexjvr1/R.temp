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









