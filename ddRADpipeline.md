#ddRAD processing pipeline  

This is the final pipeline I'm using to process the ddRAD data from raw reads to basic population statistics

##1. Quality check & filter raw data
  - FastQC check
  - Demultiplex 
      - Optimisation
      - check discard file for recovery of sequences)
  - Filter for quality
  - Filter for adapter dimer
  - Filter for read length
  - QC: nr of reads/sample (& draw graph comparison)
  
##2.1 Denovo assembly (using pyrad)
  - Optimisation of depth
  - Optimisation of clustering threshold
  - Within & between sample clustering
  - Call SNPs
  - QC: sample dropout, missingness, nr loci, comparison between lanes, comparison between populations
  
##2.2 Mapping to a genome
  - Optimisation of parameters
  - QC: mapping proportion, distribution across the genome
  - SNPcalling **Using probabilistic models 
    -FreeBayes
    -Other
  - QC: sample dropout, missingness, nr loci, comparison between lanes, comparison between populations

##3. SNP filtering (VCFtools)
  - Optimisation of parameters
    - QC: sample dropout
  - Variatinos in SNP filtering based on analyses
    - phylo, pop gts, outliers
  - QC: 
      - total nr of SNPS
      - distribution across individuals (and dropout rate)
      - SNPs across lanes (and dropout rate - correlate with library length)
      - SNPs within populations
      - SNPs across populations
      
##4. Basic statistics
  - PCA
  - Structure
  - Map of population diversity
  - HWE within populations
  - He (sliding window??)
  - Fst & Fis
  - Fst decrease with nr of samples
  - Fst/1/Fst vs Geographic distance
  

####Note

Sequencing for the *Rana temporaria* project was all conducted throught the FGCZ at UZH, Irchel. 

Contact person: Catharine Aquino (Wet lab) & Hubert Rehrauer (Bioinformatics)

Lab work was all conducted at the GDC, ETH, Zürich

Contact person: Aria Minder (Lab Manager), Silvia Skobel (Technician), & Stefan Zoller (Bioinformatcian)

All raw sequencing data is stored on the FGCZ server: /srv/gstore4users/p1795

I also have access to one of the nodes (arranged through Ken Shimitzu) for computing: /srv/kenlab/alexjvr_p1795 

        ssh alexjvr@fgcz-transfer.uzh.ch
        ssh alexjvr@fgcz-c-047
        pwd Platypu&6.

Most of the computing power is at the GDC. I have access to the servers there for data storage and computing for a nominal fee. 
        ssh alexjvr@gdcsrv1.ethz.ch
        pwd jk.W3.mm


####To secure copy files

1. From FGCZ server to the GDC server: Stefan suggests rsync for this. The following command syncs the raw data folder from FGCZ to my home directory on the GDC server 1. This is run from the FGCZ side
``` 
rsync -av -e "ssh -l alexjvr"  /srv/gstore4users/p1795 gdcsrv1.ethz.ch:/gdc_home4/alexjvr/
```

and GDC to FGCZ
```
rsync -av -e "ssh -l alexjvr"   subsetRAD_gdc/ fgcz-transfer.uzh.ch:subset_gdc
```

2. From a server onto my computer:
```
scp -r alexjvr@gdcsrv1.ethz.ch:popgts_c94d6_20150525/* .
```
3. From my computer onto the server:
```
scp -r folder/*.txt alexjvr@gdcsrv1.ethz.ch:~/
```

####Count the number of files in a folder

```
ls -l arce* | wc -l
```


####Screen options
If anything runs for more than a few mins, it is useful to run it through "screen". 
e.g. the following command will start a new screen 

-L automatically log
-S name the screen session

```
screen -S rsyncp1795 -L
```
"cntrlAD" will detach from the screen

screen help: 
```
screen man
```


  
##1. Quality check & filter raw data
  
###*FastQC check*

FastQC is a quick and easy way to check the data qualtiy as soon as it comes back from the sequencing facility. 

http://www.bioinformatics.babraham.ac.uk/projects/fastqc/

FastQC can be run directly on the raw data output from HiSeq or MiSeq. The compressed file can be used. I did this using the GUI on the mac laptop. 

Command line: 

http://www.bioinformatics.babraham.ac.uk/projects/fastqc/INSTALL.txt

Once FastQC is installed, it can be called directly and the input file specified. (I haven't done this, but got this info from the above link)
```
fastqc file.fastq.gz
```

The FGCZ ran FastQC on all my libraries. The results can be found here: 

Library | Date of run | FastQC results
:--:|:--:|:--:
H01 |28.02.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H02 |23.12.14| http://fgcz-gstore.uzh.ch/projects/p1795/HiSeq_20141219_RUN159/
H03 |28.02.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H04 |28.02.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H05 |28.02.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H06 |28.02.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H07 |28.02.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H08 |28.02.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H09 |28.02.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H10 |07.07.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H11 ||
H12 ||
H13 |07.07.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H14|07.07.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H15||
H16 |07.07.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H17 |07.07.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H18 |07.07.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H19 |07.07.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H20 ||
H21 |07.07.15|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H22 ||
H23||
H24||
H25||
H26||
H27||
H28||




###*Demultiplex*

I'm using process_radtags for demutliplexing. This is recommended by J.Puritz. 

Documentation: http://catchenlab.life.illinois.edu/stacks/comp/process_radtags.php

####1. *Optimisation*

Depending on the length of the barcode, it may make sense to allow more or less mismatches in the barcode. 
Similarly, it may make sense to include the restriction recognition site as part of the barcode to increase the length of the barcode that will be searched for. 

First:

make a directory 
```
mkdir Batch2
mkdir Batch2/barcodes_RE
mkdir Batch2/demultiplexed
```

Make the barcode files in excel, and paste into a nano file. 

E.g. H10:
```
GCATGAATT	arce_12
AACCAAATT	arce_13
CGATCAATT	ellw_05
TCGATAATT	mart_03
TGCATAATT	mctn_07
CAACCAATT	mctn_08
GGTTGAATT	mctn_09
AAGGAAATT	mctn_10
AGCTAAATT	mart_04
ACACAAATT	ellw_13
```

Then, in order to test how many mismatches are best, run the demultiplexing with 1 and with 2 mismatches (default is 1): 
```
/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150225_RUN171/20150225.A-H1_R1.fastq.gz -o ./H1 -y fastq -b ~/barcodes -e ecoRI -r -c -q -D

OR

/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150225_RUN171/20150225.A-H1_R1.fastq.gz -o ./H1 -y fastq -b ~/barcodes -e ecoRI -r -c -q -D —barcode_dist_1 2

```

In the case above, I used only the 5bp barcode, and specified the EcoRI recognition site. If the restriction site is incorporated in the barcode (to increase the length of the "barcode" to search for) the following code is used. 
```
/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150703_RUN199/20150703.A-H10_R1.fastq.gz  -o ./demultiplexed/H10 -y fastq -b ./barcodes/H10_barcodes --disable_rad_check -r -c -q -D
```



![alt txt][mm12]
[mm12]:https://cloud.githubusercontent.com/assets/12142475/8669041/60e2f46e-2a11-11e5-8567-872465ad11d4.png

      ####Figure 1: Number of reads per sample mm1 vs mm2
      If some samples are losing and others are gaining reads (as in this graph), this means reads are moving between samples.       This   should be avoided, and indicates the threshold at which mm should be set. In this case (*Rana temporaria* H01),         2mm are too    many. 


*For the Rana temporaria data set, I will use 5bp barcode + Restriction recognition site (9bp total) for demultiplexing, and allow 1mm.* 


####2. *Demultiplex samples using 5bp barcode + RE*

(As before) Make a directory 
```
mkdir Batch2
mkdir Batch2/barcodes_RE
mkdir Batch2/demultiplexed
```

Make the barcode files in excel, and paste into a nano file. 

E.g. H10:
```
GCATGAATT	arce_12
AACCAAATT	arce_13
CGATCAATT	ellw_05
TCGATAATT	mart_03
TGCATAATT	mctn_07
CAACCAATT	mctn_08
GGTTGAATT	mctn_09
AAGGAAATT	mctn_10
AGCTAAATT	mart_04
ACACAAATT	ellw_13
```


Demultiplex using the output from the HiSeq. 
--disable_rad_check will allow the inclusion of the restriction site in the barcode

-r : rescue barcodes and radtags

-c : clean data, remove and read with an uncalled base

-q : discard reads with low quality scores (Q20)

-D : capture discarded reads to a file


For *R.temp* data I ran process_radtags (only -r: rescue radtags. -c -q filters removed, since data will be cleaned downstream): 

```
/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150703_RUN199/20150703.A-H10_R1.fastq.gz  -o ./demultiplexed/H10 -y fastq -b ./barcodes/H10_barcodes --disable_rad_check -r -D
```



Results from *R.temp* libraries Dec 2014-2015 (without any filters)

Library | Total Reads |%PhiX| Ambiguous Barcode drops |%| AdapterDimer |%| Retained reads|%
:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:
H01|251376944|20%|79670378|31.7%|||171706566|
H02|215305263|20%|47501389|22.1%|||167803874|
H03|246670582|20%|80666381|32.7%|||166004201|
H04|240547023|20%|85943740|35.7%|||154603283|
H05|256120714|20%|85670480|33.4%|||170450234|
H06|249625282|20%|84319210|33.8%|||165306072|
H07|242959478|20%|85573060|35.2%|||157386418|
H08|251997528|20%|90482676|35.9%|||161514852|
H09|251280205|20%|88708627|35.3%|||162571578|
H10|103020694|15%|23130351|22.5%||| 79890343|
H11||||||||
H12||||||||
H13|218264182|15%|72124541|33%  |||146139641|
H14|217765320|15%|65872858|30.2%|||151892462|
H15||||||||
H16|237732001|15%|70441917|29.6%|||167290084|
H17|220987734|15%|67899013|30.7%|||153088721|
H18|222052040|15%|66204976|29.8%|||155847064|
H19|189469463|15%|53558760|28.2%|||135910703|
H20||||||||
H21|104792211|15%|27781697|26.5%||| 77010514|
H22| 84372280|15%|24013662|28.5%||| 54967074|
H23|151374953|15%|48120207|31.8%||| 98293089|
H24||||||||
H25||||||||
H26||||||||
H27||||||||
H28||||||||



####3. *Check discard file for recovery of sequences*

If there is an insertion at the start of the sequence, the demultiplexing algorith will not recognise the barcode, and discard it. (Debbie Leigh found this in her data set). 

Check whether there are obvious problems causing usable data to be filtered. 

This is particularly important if a much larger proportion of sequences are filtered for "ambiguous barcodes" than expected from the amount of PhiX added. 


####4. *Filter for quality*

The quality filter is not really necessary at this point, and can instead be done during mapping/de novo assembly or even SNP calling. This approach is recommmended by some (e.g. Christine Grossen), so that as much of the data as possible is used. Filtering after SNP calling is faster and gives more control over the ready data sets. 

Here I've filtered for Q20 because the data overall looks really good. I'm filtering <4% of the reads out in this step. 


####5. *Filter for adapter dimer*

This is really important, even if the BioAnalyzer peak looks clean. Adapters can form very large dimers that co-migrate with the main library. 

I am using AdapterRemoval, as recommended by Christine Grossen

AdapterRemoval: https://github.com/slindgreen/AdapterRemoval

```
adapterremoval --file1 csee_02.fq --basename AdRem --trimns --trimqualities --minquality 20 --adapter1 AATGATACGGCGACCACCGAGATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT
```

And batch script: 
```
for x in *.fq
do adapterremoval --file1 ${x} --basename AdRem_${x} --trimns --trimqualities --minquality 20 --adapter1 AATGATACGGCGACCACCGAGATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT
done
```

But, neither FGCZ or GDC have AdapterRemoval installed. 

FGCZ (Hubert) suggests Trimmomatic

Location on fgcz server
```
java -jar /usr/local/ngseq/src/Trimmomatic-0.33/trimmomatic-0.33.jar
```

and on the gdc server
```
java -jar /usr/local/trimmomatic/trimmomatic-0.32.jar 
```

General code:
```
java -jar <path to trimmomatic jar> SE [-threads <threads>] [-phred33 |-phred64] [-trimlog<logFile>] <input> <output> <step 1> 

```
And manual: http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf

Code I used on the gdcsrv for the subset data

```
java -jar /usr/local/trimmomatic/trimmomatic-0.32.jar SE shwe_01.trimlog shwe_01.fq shwe_01.trim.fq ILLUMINACLIP:TruSeq3-SE:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36 
```

- Phred encoding of my data is Phred33

- I'm using TruSeq3 adapters (with Trimmomatic software) which is used in Illumina HiSeq SE sequencing

Batch script to run this on all samples and save sequences in a new folder

Run from the folder containing the .fq files. 
Run in screen and log output

```
screen -S TrimSubset -L

for i in *.fq; do  java -jar /usr/local/trimmomatic/trimmomatic-0.32.jar SE $i $i.trim ILLUMINACLIP:/usr/local/trimmomatic/adapters/TruSeq3-SE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36; done
```

Copy over to a new folder 
```
mv *.trim pyrad/subsetTrim
```



####6. *Filter for read length*


This can be done within process_radtags, using the -t option


####7. *QC*


  - QC: nr of reads/sample (& draw graph comparison)
  
1. Bar graph of the number of reads per library

Excel input for my data is called: RAD_QC_Rawreads_20150722.csv


And the input looks like this: 

![alt txt][input]
[input]:https://cloud.githubusercontent.com/assets/12142475/8831179/e9b70664-30a0-11e5-8364-7320fc1b5ff7.png


R-code. las turns the labels perpendicular. cex.names changes the font size.
```
Table <- RAD_QC_Rawreads_20150722
summary(Table)
barplot(Table$RawReadsMil, names=Table$Library, main="Number of raw reads per Illumina HiSeq library", xlab="Library", ylab="Read Count (Million)", cex.names=0.8, las=2)
```

#####Figure3 
Number of raw reads per library

![alt txt][rawreads]
[rawreads]:https://cloud.githubusercontent.com/assets/12142475/8831205/12e8a4fc-30a1-11e5-88b0-d0e9cd066778.png


2. Bar graph of average & SD of reads per barcode across libraries

Draw a bar graph in R. 
Data file should look like this: 

R code: 
```
```

My data: 




#####Figure4 
Average and range of the number of final reads per population

- These are the final reads after demultiplexing and Trimmomatic

Input data file



R code: 
- varwidth changes bar width accordign to the number of samples
- par changes font size (cex.axis or cex.ylab..)
- las changes orientation of x-axis labels

```
#Boxplot for reads per population

#short name for the data matrix
subset <-RADsubset_Trimmedreads_Milmore
summary(subset)

#write to png
png("Subset_avgreads.png")

#plot. varwidth changes the barplot width according to the number of samples
boxplot(subset$RetainedMil~subset$Pop, data=subset, varwidth=T, par(cex.axis=0.8), las=2, main="Average reads per population (Million)", xlab="population", ylab="Reads (million)")

#write to png(end)

dev.off()
```


3. Bar graph of low quality reads dropped

R code: 
```
```

My data: 




#####Figure5
% reads filtered during demultiplexing. This is due to lack of barcode (PhiX) or ambiguous barcodes (too many mismatches), or quality (Phred <Q20)


##Data Subset

At this point, I have decided to use only a subset of the data to set up the rest of the pipeline and run through the rest of the analyses. 

The following is a map of the chosen sites, and a table with some information for each site. 

![alt txt][RAD_subset]
[RAD_subset]:https://cloud.githubusercontent.com/assets/12142475/8824708/94418bfa-3078-11e5-8bbd-c087297f808a.png


Canton|Site|Name|Lat|Long|Elevation|n|libraries
:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:
BE|kand|Kanderauen bei Mülenen	|46.63368|	7.69037|698|11|H14
BE|tsee|Tschingelsee	|46.55400|	7.74653|1151|18|H16 H17
BE|meie|Meiefallsseeli	|46.58680|	7.58405|1900|15|H10 H17 H18 H19 H21
SG|wise|Wisenfurt	|47.19467	|9.47862|445|6|H13
SG|shwe|Schwendiseen|	47.18790|	9.32981|1159|10|H01 H02
SG|bide|Bi den Seen	|47.14247	|9.40407|1983|20|H05
VS|orge|Lac de Mont dOrge|	46.23357|	7.33560|643|11|H10 H13 H21
VS|lens|Etang de Lens|	46.29206	|7.45947|1338|19|H18 H19
VS|csan|Col du Sanetsch|	46.32276|	7.30013|2110|20|H18
TI|arce|Arcegno Brumo |	46.15637	|8.74613|408|13|H10 H13 H21
TI|gola|Gola di Lago|	46.10463	|8.96543|975|12|H13 H21
TI|star|Lago del Starlaresc da Sgiof	|46.27347|	8.77321|1892|20|H07


Distribution of the final numbers of reads per population

- Ive removed samples with <1Mil reads
- removed samples: kand_03, lens_06, meie_14, orge_03-05, wise_03 


![alt.txt][data]
[data]:https://cloud.githubusercontent.com/assets/12142475/8852524/05e79870-3156-11e5-9edb-4f032c38175c.png

![alt.txt][subset_avgreads]
[subset_avgreads]:https://cloud.githubusercontent.com/assets/12142475/8852785/6f951620-3157-11e5-8a7e-01a9be7a727b.png




##2.1 Denovo assembly (using pyrad)

Location of pyrad: 

- GDC: /usr/local/pyrad-3.0.6/pyrad

- FGCZ: /usr/local/ngseq/src/pyrad-3.0.4/pyRAD

First a params.txt file needs to be created. 

Important things to note: 

- indicate the location of vsearch and muscle
- if samples are demultiplexed, start at s2 & indicate location of demultiplexed data
- par 13 is mandatory. Set this really high

Basic code to start a run (using parameters optimised for *R.temp* data H02)

1. create a params.txt file
```
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -n
```

start the run
```
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 23456
```

params.txt file as optimised for H02 *R.temp* data. 

- Im including singletons (#34) for comparison with R.temp genome

```
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
.94                       ## 10. Wclust: clustering threshold as a decimal        (s3,s6)
ddrad                       ## 11. Datatype: rad,gbs,pairgbs,pairddrad,(others:see docs)(all)
50                         ## 12. MinCov: min samples in a final locus             (s7)
200                         ## 13. MaxSH: max inds with shared hetero site          (s7)
c96d6m4min50                 ## 14. Prefix name for final output (no spaces)         (s7)
==== optional params below this line ===================================  affected step ==
                       ## 15.opt.: select subset (prefix* only selector)            (s2-s7)
                       ## 16.opt.: add-on (outgroup) taxa (list or prefix*)         (s6,s7)
                       ## 17.opt.: exclude taxa (list or prefix*)                   (s7)
./subsetTrim/*.fq.trim                       ## 18.opt.: loc. of de-multiplexed data                      (s2)
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
     *                  ## 30.opt.: output formats: p,n,a,s,v,u,t,m,k,g,* (see docs) (s7)
                       ## 31.opt.: maj. base call at depth>x<mindepth (def.x=mindepth) (s5)
    50                   ## 32.opt.: keep trimmed reads (def=0). Enter min length.    (s2)
                       ## 33.opt.: max stack size (int), def= max(500,mean+2*SD)    (s3)
                       ## 34.opt.: minDerep: exclude dereps with <= N copies, def=1 (s3)
                       ## 35.opt.: use hierarchical clustering (def.=0, 1=yes)      (s6)
                       ## 36.opt.: repeat masking (def.=1='dust' method, 0=no)      (s3,s6)
                       ## 37.opt.: vsearch max threads per job (def.=6; see docs)   (s3,s6)
==== optional: list group/clade assignments below this line (see docs) ==================
```

#####*run for subset started 23 July 2015, 17:14. This includes 175 individuals*


Average number of retained reads per population after initial cleanup (demultiplex & adapter removal)

https://cloud.githubusercontent.com/assets/12142475/8907222/e6c8823e-3474-11e5-9246-01635ab3e724.png


Average number of within sample clusters from PyRAD output: 

https://cloud.githubusercontent.com/assets/12142475/8907229/f0692474-3474-11e5-976a-1a7d370ebe38.png

####1. *Optimisation of depth*
####2. *Optimisation of clustering threshold*
####3. *Within & between sample clustering*
####4. *Call SNPs*
####5. *QC* 

sample dropout, missingness, nr loci, comparison between lanes, comparison between populations
  
  
  
  
##2.2 Mapping to a genome

- Nice treatment on mapping best practices: http://cgrlucb.wikispaces.com/file/view/readMapping.pdf

Received the R.temporaria draft genome from Alan Brelsford. This is the information he sent me:

*The genome itself is pretty rough; it amounts to 35% of the expected genome size, and the scaffold n50 is only 924 bp. On the other hand, 98% of the CEGMA core genes are at least partially represented, so I think it includes most of the non-repetitive part of the genome, and the missing 3 Gbp is enriched for repeats. About 10% of the genome assembly can be assigned to a position in X. tropicalis.

Assembling the genome took quite a bit of time and effort, so if you decide to use it, I would like to be included as a coauthor on one paper. Is this something you and Josh are open to?*

File name:
Rtk43.fa.gz (473Mb)


####bowtie2

Im using bowtie2 as an aligner, since it incorporates base quality in when mapping. See Hatem et al. 2013 for a treatment on different aligners. 

- Manual: http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml

- Tutorial: http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml#performance-tuning



####Basic pipeline starting from demultiplexed sequences with adapter dimer removed: 

1. index the genome. 

This only needs to be done once. This takes ~(15:24-

On the fgcz server: bowtie2-build <input reference (must be in fasta)> <prefix of output> 
```
/usr/local/ngseq/bin/bowtie2-build Rtk43.fa Rtk43
```

Output (for future use): 
/srv/kenlab/alexjvr_p1795/ddRADpipeline/mapping/Rtk43.*

Rtk43.1.bt2  Rtk43.2.bt2  Rtk43.3.bt2  Rtk43.4.bt2  Rtk43.rev.1.bt2  Rtk43.rev.2.bt2

/srv/kenlab/alexjvr_p1795/ddRADpipeline/mapping/Rtk43.fa (1.8Gb)

2. 

####1. *Optimisation of parameters*

See here for a good tutorial: https://wikis.utexas.edu/display/bioiteam/Mapping+with+bowtie2+Tutorial#Mappingwithbowtie2Tutorial-Mappingtoolssummary

- For optimisation, use the fastq file of 3 samples (cleaned & trimmed), and vary the parameters. To start, use default parameters for the --sensitive analysis

- Evaluate the run based on the number and quality of reads output from each. 

- The most important parameter to vary is the number of nucleotide differences allowed. This should match the expected number of differences between two sequences from your species. 

parameters to vary with the bowtie2 algorithm: 

-D #seed extensions that can fail before moving on. (10;15;20)

-R # of re-seeding attempts  (1;2;3)

-N number of mismatches in seed alignment (0;1)

-L length of the seed (15;22;50)

- The goal is to maximise the number of reads mapped singly, without increasing the mapping time to much. 

####Optimisation 

csan_05.fq.trim & meie_08.fq.trim for the optimisation

1. Parameter varied: -L (seed length) 15;22;32 (max allowed is 32)
```
/usr/local/ngseq/bin/bowtie2 -x Rtk43 -U /srv/kenlab/alexjvr_p1795/ddRADpipeline/subsetTrim/csan_05.fq.trim --phred33 --end-to-end --time --threads 4 -D 15 -R 2 -N 0 -i S,1,1.15 -L 15 -S csan_05L15.sam  
```

Parameter value| total reads | unmapped | mapped once | aligned >1| overall alignment rate | time
:---:|:---:|:---:|:---:|:---:|:---:|:---:
15|3926405|2220143 (56.54%)|1199346 (30.55%)|506916 (12.91%)|43.46%|13:27
22|3926405 |2071569 (52.76%)|1260294 (32.10%)|594542 (15.14%)|47.24%|9:36
32|3926405| 2352654 (59.92%)|1172575 (29.86%)|401176 (10.22%)|40.08%|6:04

2. Parameter varied: -N (#mismatches in seed alignment) 0;1
```
/usr/local/ngseq/bin/bowtie2 -x Rtk43 -U /srv/kenlab/alexjvr_p1795/ddRADpipeline/subsetTrim/csan_05.fq.trim --phred33 --end-to-end --time --threads 4 -D 15 -R 2 -N 1 -i S,1,1.15 -L 22 -S csan_05L22N0.sam  
```

Parameter value| total reads | unmapped | mapped once | aligned >1| overall alignment rate | time
:---:|:---:|:---:|:---:|:---:|:---:|:---:
L15N1|3926405|3068902 (78.16%)|733650 (18.69%)|123853 (3.15%)|21.84%|01:12:13
L15N0|3926405|2220143 (56.54%)|1199346 (30.55%)|506916 (12.91%)|43.46%|13:27
L22N1|3926405| 2071406 (52.76%)|1260226 (32.10%)|594773 (15.15%)|47.24%|10:40
L22N0|3926405 |2071569 (52.76%)|1260294 (32.10%)|594542 (15.14%)|47.24%|9:13




3. Parameter varied: -R (#re-seeding attempts for repetitive seeds) 1;2;3 (L22N0)
```
/usr/local/ngseq/bin/bowtie2 -x /srv/kenlab/alexjvr_p1795/ddRADpipeline/mapping/Rtk43 -U /srv/kenlab/alexjvr_p1795/ddRADpipeline/subsetTrim/csan_05.fq.trim --phred33 --end-to-end --time --threads 4 -D 15 -R 2 -N 0 -i S,1,1.15 -L 22 -S csan_05L22N0R2.sam
```

Parameter value| total reads | unmapped | mapped once | aligned >1| overall alignment rate | time
:---:|:---:|:---:|:---:|:---:|:---:|:---:
L22N0R1|3926405|2072040 (52.77%)|1260170 (32.09%)|594195 (15.13%)|47.23% |00:08:33
L22N0R2|3926405 |2071569 (52.76%)|1260294 (32.10%)|594542 (15.14%)|47.24%|9:13
L22N0R3|3926405 reads| 2071406 (52.76%)| 1260226 (32.10%)|594773 (15.15%)|47.24%|10:30


4. Parameter varied: -D (#seed extensions that can fail before moving on.) 10;15;20 (L22N0R2)
```
/usr/local/ngseq/bin/bowtie2 -x /srv/kenlab/alexjvr_p1795/ddRADpipeline/mapping/Rtk43 -U /srv/kenlab/alexjvr_p1795/ddRADpipeline/subsetTrim/meie_08.fq.trim --phred33 --end-to-end --time --threads 4 -D 10 -R 2 -N 0 -i S,1,1.15 -L 22 -S meie_08L22N0R2D10.sam
```

Parameter value| Sample|total reads | unmapped | mapped once | aligned >1| overall alignment rate | time
:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:
L22N0R2D10|csan_05|3926405|2173182 (55.35%)|1261317 (32.12%)|491906 (12.53%)|44.65%|08:15
L22N0R2D10|meie_08|3428077| 1920452 (56.02%)| 1066464 (31.11%)|441161 (12.87%)|43.98%|07:36
L22N0R2D15|csan_05|3926405 |2071569 (52.76%)|1260294 (32.10%)|594542 (15.14%)|47.24%|9:13
L22N0R2D15|meie_08|3428077|1824343 (53.22%)|1066389 (31.11%)|537345 (15.67%)|46.78% |09:13
L22N0R2D20|csan_05|3926405| 2010770 (51.21%)|1254768 (31.96%)|660867 (16.83%)|48.79%|11:37
L22N0R2D20|meie_08||||||




Final optimised parameters: 
```
/usr/local/ngseq/bin/bowtie2 -x /srv/kenlab/alexjvr_p1795/ddRADpipeline/mapping/Rtk43 -U /srv/kenlab/alexjvr_p1795/ddRADpipeline/subsetTrim/csan_05.fq.trim --phred33 --end-to-end --time --threads 4 -D 15 -R 2 -N 0 -i S,1,1.15 -L 22 -S csan_05L22N0R2D15.sam
```
This is the same as the --sensitive setting in the --end-to-end mode, so the final script can be for aligning multiple reads:  

```
/usr/local/ngseq/bin/bowtie2 -x /srv/kenlab/alexjvr_p1795/ddRADpipeline/mapping/Rtk43 -U /srv/kenlab/alexjvr_p1795/ddRADpipeline/subsetTrim/*.fq.trim --phred33 --end-to-end --time --threads 4 --sensitive -S *.sam 
```

To view the output:

- the .sam files are not all that useful to look at, so we need to process them with samtools

1. Index the genome with samtools (needs it's own indexed genome) - this takes about 1min, but only needs to be done once

```
/usr/local/ngseq/bin/samtools faidx  /srv/kenlab/alexjvr_p1795/ddRADpipeline/mapping/Rtk43.fa
```


2. Next, we need to convert the SAM file into a BAM file. (A BAM file is just a binary version of a SAM file.) - ~2min on fgcz server
```
/usr/local/ngseq/bin/samtools import Rtk43.fa.fai csan_05.sam csan_05.bam
```

3. sort the .bam file

```
/usr/local/ngseq/bin/samtools sort csan_05.bam csan_05.bam.sorted
```

4. Index the sorted .bam file. Output *.bam.bai

```
/usr/local/ngseq/bin/samtools index csan_05.bam.sorted.bam
```

Now we can look at the data: 

```
/usr/local/ngseq/bin/samtools flagstat csan_05.bam.sorted.bam
```




####2. *QC*

  mapping proportion, distribution across the genome



bar graph: number of sequences mapped per sample

bar graph: sequences mapped per population
          proportion of mapped reads per population




####3. *SNPcalling*
**Using probabilistic models 
    -FreeBayes


Tutorial and information here: http://clavius.bc.edu/~erik/CSHL-advanced-sequencing/freebayes-tutorial.html 

Freebayes is not on the fgcz server, but is on gdc server:
```
/usr/local/freebayes/bin/freebayes
```
- need an uncompressed reference genome in fasta format
- alignment files (BAM format)


Run freebayes:
```
/usr/local/freebayes/bin/freebayes -f <genome.fa> \ <input.bam> > <output.vcf>
```

Look at the output: 
```
/usr/local/freebayes/bin/vcfstats NA12878.chr20.freebayes.vcf
```


Parameters to optimise: 

- 

- 

- 





####4. *QC*
sample dropout, missingness, nr loci, comparison between lanes, comparison between populations


  
##3. SNP filtering (VCFtools)

####1. *Optimisation of parameters*
####2. *QC: sample dropout*

####3. *Variatinos in SNP filtering based on analyses*
    - phylo, pop gts, outliers

####4. *QC*: 
      - total nr of SNPS
      - distribution across individuals (and dropout rate)
      - SNPs across lanes (and dropout rate - correlate with library length)
      - SNPs within populations
      - SNPs across populations



##4. Basic statistics
  - PCA
  - Structure
  - Map of population diversity
  - HWE within populations
  - He (sliding window??)
  - Fst & Fis
  - Fst decrease with nr of samples
  - Fst/1/Fst vs Geographic distance
