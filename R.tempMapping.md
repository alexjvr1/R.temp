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
This is for all 451 individuals. 
Start 23:52
```
bwa aln Rtk43.fa demultiplexed/*.fq.gz > Rt.*.sai
```

Convert to sam format
```
bwa samse Rtk43.fa Rt.abnd09.sai demultiplexed/abnd_09_H5.fq.gz > Rt.abnd09.sai.sam

#I got an error message when running this for all abnd01-09. And all the files are exactly the same size. I'm not sure whether this is right or not

[bwa_aln_core] convert to sequence coordinate... 3.17 sec
[bwa_aln_core] refine gapped alignments... 0.89 sec
[bwa_aln_core] print alignments... 0.56 sec
[bwa_aln_core] 262144 sequences have been processed.
[bwa_aln_core] convert to sequence coordinate... 3.11 sec
[bwa_aln_core] refine gapped alignments... 0.89 sec
[bwa_aln_core] print alignments... 0.55 sec
[bwa_aln_core] 524288 sequences have been processed.
[fread] Unexpected end of file
```

Now I have to use samtools

First, index the genome (this only needs to be done once) (~2min)
```
samtools faidx Rtk43.fa
```

Next, convert SAM to BAM files. (A BAM file is just a binary version of a SAM files) (1min)
```
samtools import Rtk43.fa.fai Rt.abnd01.sai.sam Rt.abnd01.sai.bam
```

And then sort the BAM file (1min)
```
samtools sort Rt.H2.bam Rt.H2.bam.sorted
```

And last, we need Samtools to index the BAM file (10s)
```
samtools index Rt.H2.bam.sorted.bam
```

Now I need to check the output of the alignments. 
1. What proportion of the sequences mapped?
2. What's the comparison between samples?
3. 

For basic statistics use the flagstat command
```
samtools flagstat Rt.abnd03.sai.bam.sorted.bam
```

OR
the idxstats command
```
samtools idxstats Rt.abnd03.sai.bam.sorted.bam
```

###Problem
I'm only getting ~100 000 sequences mapped. For some reason only ~500 000 reads were used to start with (this is already 1/7 of the 3.5mil reads). And then I'm only mapping 19% of these reads..   

So I need to relax the initial parameters.

BUT my demultiplexed files seem much smaller than they're supposed to be. I should have ~3.5mil reads per sample, but there is only ~500 000 in each. So on top of getting a low mapping success, I'm starting with far fewer reads than I should! What went wrong with the demultiplexing? I will have to redo this step to check what the problem is. 

And then I need to call SNPs

I will use FreeBayes


