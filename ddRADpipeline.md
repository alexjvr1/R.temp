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
  

*Demultiplex*

1. Optimisation

Depending on the length of the barcode, it may make sense to allow more or less mismatches in the barcode. 
Similarly, it may make sense to include the restriction recognition site as part of the barcode to increase the length of the barcode that will be searched for. 

In order to test how many mismatches are best, run the demultiplexing with 1 and with 2 mismatches: 
```
```


For the *Rana temporaria* data set, the optimal is to allow 1 mismatch in the barcode + Restriction site: 

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
