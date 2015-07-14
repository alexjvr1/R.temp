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

H2: http://fgcz-gstore.uzh.ch/projects/p1795/HiSeq_20141219_RUN159/

H1-9: http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc_5027_2015-03-06--09-42-52/FastQC_Result/00index.html

H10,H13-14,H16-19,H21: http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/


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

Then, in order to test how many mismatches are best, run the demultiplexing with 1 and with 2 mismatches: 
```
/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150225_RUN171/20150225.A-H1_R1.fastq.gz -o ./H1 -y fastq -b ~/barcodes -e ecoRI -r -c -q -D

OR


```

In the case above, I used only the 5bp barcode, and specified the EcoRI recognition site. If the restriction site is incorporated in the barcode (to increase the length of the "barcode" to search for) the following code is used. 
```
/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150703_RUN199/20150703.A-H10_R1.fastq.gz  -o ./demultiplexed/H10 -y fastq -b ./barcodes/H10_barcodes --disable_rad_check -r -c -q -D
```


Plot number of reads per sample mm1 vs mm2

If some samples are losing and others are gaining reads, this means reads are moving between samples. 


For the *Rana temporaria* data set, the optimal is to allow 1 mismatch in the barcode + Restriction site: 

- graph of mm1 vs mm2

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
```
/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150703_RUN199/20150703.A-H10_R1.fastq.gz  -o ./demultiplexed/H10 -y fastq -b ./barcodes/H10_barcodes --disable_rad_check -r -c -q -D
```

Results from *R.temp* libraries 2014-2015

Library | Total Reads |%PhiX| Ambiguous Barcode drops |%| Low-quality reads dropped |%| Retained reads|%
:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:
H01|251376944|20%|||||161591671|64.3%
H02|215305263|20%|||||162140179|75.3%
H03|246670582|20%|||||156520226|63.5%
H04|240547023|20%|||||146085735|60.7%
H05|256120714|20%|||||159781351|62.4%
H06|249625282|20%|||||156589851|62.7%
H07|242959478|20%|||||148479256|61.1%
H08|251997528|20%|||||152588999|60.6%
H09|251280205|20%|||||152305138|60.6%
H10 | 103020694 |15%| 23130351|22.5%|11076359|10.8%|68813984|66.8%
H11||||||||
H12||||||||
H13 |218264182|15%|72124541|33%|5463292|2.5%|140676349|64.5%
H14 |217765320|15%|65872858|30.2%|8037169|3.7%|143855293|66.1%
H15||||||||
H16|237732001|15%||||||
H17|220987734|15%||||||
H18|222052040|15%||||||
H19|189469463|15%||||||
H20||||||||
H21|104792211|15%|27781697|26.5%|10978916|10.5%|66031598|63.0%
H22||||||||
H23||||||||
H24||||||||
H25||||||||
H26||||||||
H27||||||||
H28||||||||



####3. *Check discard file for recovery of sequences*


####4. *Filter for quality*


####5. *Filter for adapter dimer*


####6. *Filter for read length*


####7. *QC*


  - QC: nr of reads/sample (& draw graph comparison)
  
1. Bar graph of the number of reads per library
2. Bar graph of average & SD of reads per barcode across libraries
3. Bar graph of low quality reads dropped



##2.1 Denovo assembly (using pyrad)

####1. *Optimisation of depth*
####2. *Optimisation of clustering threshold*
####3. *Within & between sample clustering*
####4. *Call SNPs*
####5. *QC* 

sample dropout, missingness, nr loci, comparison between lanes, comparison between populations
  
  
  
  
##2.2 Mapping to a genome

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
