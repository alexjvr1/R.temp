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
2. From a server onto my computer:
```
scp -r alexjvr@gdcsrv1.ethz.ch:popgts_c94d6_20150525/* .
```
3. From my computer onto the server:
```
scp -r folder/*.txt alexjvr@gdcsrv1.ethz.ch:~/
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
H01 |28 Feb 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H02 |23 Dec 2014| http://fgcz-gstore.uzh.ch/projects/p1795/HiSeq_20141219_RUN159/
H03 |28 Feb 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H04 |28 Feb 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H05 |28 Feb 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H06 |28 Feb 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H07 |28 Feb 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H08 |28 Feb 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H09 |28 Feb 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html
H10 |7 Jul 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H11 ||
H12 ||
H13 |7 Jul 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H14|7 Jul 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H15||
H16 |7 Jul 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H17 |7 Jul 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H18 |7 Jul 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H19 |7 Jul 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
H20 ||
H21 |7 Jul 2015|http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/
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


####6. *Filter for read length*


This can be done within process_radtags, using the -t option


####7. *QC*


  - QC: nr of reads/sample (& draw graph comparison)
  
1. Bar graph of the number of reads per library


R code: 
```
```

My data: 




#####Figure3 
Number of raw reads per library

2. Bar graph of average & SD of reads per barcode across libraries

Draw a bar graph in R. 
Data file should look like this: 

R code: 
```
```

My data: 




#####Figure4 
Average and range of the number of reads per sample per library


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

##2.1 Denovo assembly (using pyrad)

####1. *Optimisation of depth*
####2. *Optimisation of clustering threshold*
####3. *Within & between sample clustering*
####4. *Call SNPs*
####5. *QC* 

sample dropout, missingness, nr loci, comparison between lanes, comparison between populations
  
  
  
  
##2.2 Mapping to a genome


Received the R.temporaria draft genome from Alan Brelsford. This is the information he sent me:

*The genome itself is pretty rough; it amounts to 35% of the expected genome size, and the scaffold n50 is only 924 bp. On the other hand, 98% of the CEGMA core genes are at least partially represented, so I think it includes most of the non-repetitive part of the genome, and the missing 3 Gbp is enriched for repeats. About 10% of the genome assembly can be assigned to a position in X. tropicalis.

Assembling the genome took quite a bit of time and effort, so if you decide to use it, I would like to be included as a coauthor on one paper. Is this something you and Josh are open to?*

File name:
Rtk43.fa.gz (473Mb)

Basic pipeline starting from demultiplexed sequences with adapter dimer removed: 

1. index the genome. 

This only needs to be done once. 

```
bwa index Rtk43.fa.gz
```

2. 

####1. *Optimisation of parameters*




####2. *QC*
    mapping proportion, distribution across the genome



####3. *SNPcalling*
**Using probabilistic models 
    -FreeBayes
    -Other




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
