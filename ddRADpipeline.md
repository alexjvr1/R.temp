#ddRAD processing pipeline  

This is the final pipeline I'm using to process the ddRAD data from raw reads to basic population statistics

###1. Quality check & filter raw data
  - FastQC check
  - Demultiplex 
      - Optimisation
      - check discard file for recovery of sequences)
  - Filter for quality
  - Filter for adapter dimer
  - Filter for read length
  - QC: nr of reads/sample (& draw graph comparison)
  
###2.1 Denovo assembly (using pyrad)
  - Optimisation of depth
  - Optimisation of clustering threshold
  - Within & between sample clustering
  - Call SNPs
  - QC: sample dropout, missingness, nr loci, comparison between lanes, comparison between populations
  
###2.2 Mapping to a genome
  - Optimisation of parameters
  - QC: mapping proportion, distribution across the genome
  - SNPcalling **Using probabilistic models 
    -FreeBayes
    -Other
  - QC: sample dropout, missingness, nr loci, comparison between lanes, comparison between populations
  
###3. SNP filtering (VCFtools)
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
      
###4. Basic statistics
  - PCA
  - Structure
  - Map of population diversity
  - HWE within populations
  - He (sliding window??)
  - Fst & Fis
  - Fst decrease with nr of samples
  - Fst/1/Fst vs Geographic distance
  
  
  
  
##1. Quality check & filter raw data
  
*FastQC check*

FastQC can be run directly on the raw data output from HiSeq or MiSeq. The compressed file can be used. 
```
```



*Demultiplex*

1. Optimisation

Depending on the length of the barcode, it may make sense to allow more or less mismatches in the barcode. 
Similarly, it may make sense to include the restriction recognition site as part of the barcode to increase the length of the barcode that will be searched for. 

In order to test how many mismatches are best, run the demultiplexing with 1 and with 2 mismatches: 
```
```

Plot number of reads per sample mm1 vs mm2

If some samples are losing and others are gaining reads, this means reads are moving between samples. 


For the *Rana temporaria* data set, the optimal is to allow 1 mismatch in the barcode + Restriction site: 

- graph of mm1 vs mm2

Demultiplex samples using 5bp barcode + RE

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


Demultiplex using the output from the HiSeq. 
--disable_rad_check will allow the inclusion of the restriction site in the barcode
```
/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150703_RUN199/20150703.A-H10_R1.fastq.gz  -o ./demultiplexed/H10 -y fastq -b ./barcodes/H10_barcodes --disable_rad_check -r -c -q -D
```

Results from *R.temp* libraries

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




      - check discard file for recovery of sequences)



  - Filter for quality
  - Filter for adapter dimer
  - Filter for read length
  - QC: nr of reads/sample (& draw graph comparison)
  
1. Bar graph of the number of reads per library
2. Bar graph of average & SD of reads per barcode across libraries
3. Bar graph of low quality reads dropped



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
