#R.temp Mapping

I am optimising the mapping and SNP calling protocol for R.temporaria using the draft genome from Alan Brelsford. 
The deNovo assembly takes >1 day per lane to run, and needs to be rerun as samples are added to the dataset. 

SNP calling when mapping is possible using low coverage sequencing when using a probabilistic approch to SNP calling. So this is perfect for the *Rana temporaria* data.

1. Mapping quality
2. SNP calling
3. SNP filtering
4. Comparison with deNovo assembly
  - how much data is used/lost?
  - how many SNPs are called?
  - What is the overlap in SNPs between HiSeq runs
  - Time taken for SNP calling
5. Summary statistics


##Pipeline

Overview of a pipeline is here: 

http://www.ebi.ac.uk/training/sites/ebi.ac.uk.training/files/materials/2014/140217_AgriOmics/dan_bolser_snp_calling.pdf

1. Align
  - Index genome
  - 
2. SNP calling
3. Filter SNPs

###1. Align to the genome

Start 13:24 end 14:37 
```
bwa index Rtk43.fa
```
output: 
Rtk43.fa.amb  Rtk43.fa.ann  Rtk43.fa.bwt  Rtk43.fa.pac  Rtk43.fa.sa



Align the demultiplexed individuals to the indexed genome. 
This is for one individual
Start: 14:50 end 23:52
```
bwa aln Rtk43.fa demultiplexed/abnd_01_H4.fq.gz > Rt.abnd01.sai
```

To align all the individuals
```
```

