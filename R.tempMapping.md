#R.temp Mapping

##Rerun April 2016

I am retrying the mapping at UCLA with both Alan's genome and the newly available transcriptome. 

I am using the SE dataset for the initial run. 

###Mapping to Alan's R.temp genome

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

**use bwa-mem

1. Align
  - Index genome
2. SNP calling
3. Filter SNPs

###1. Align to the genome

1hour on GDCsrv1
```
bwa index Rtk43.fa
```
output: 
Rtk43.fa.amb  Rtk43.fa.ann  Rtk43.fa.bwt  Rtk43.fa.pac  Rtk43.fa.sa



Align the demultiplexed individuals to the indexed genome. 
This is for one individual
~108min
```
bwa aln Rtk43.fa.gz /gdc_home4/alexjvr/SEFinalSamples/SEsamples_193.19/F21.fq.trim > Rt.F21.sai
```

To align all the individuals
This is for all 451 individuals. 
Start 23:52
```
bwa aln Rtk43.fa demultiplexed/*.fq.gz > Rt.*.sai
```

Convert to sam format
```
bwa samse Rtk43.fa.gz Rt.abnd09.sai demultiplexed/abnd_09_H5.fq.gz > Rt.abnd09.sai.sam

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
samtools sort Rt.H2.bam -o Rt.H2.bam.sorted
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

###Results

I used the demultiplexed & trimmed reads from both of the following samples: 

SE data (F21)

```
samtools flagstat Rt.F21.bam.sorted

2374365 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
420707 + 0 mapped (17.72% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```

And on one of the CH data (vora_01)

```
samtools flagstat Rt.vora01.bam.sorted

3758034 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
795577 + 0 mapped (21.17% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```


###Mapping to the transcriptome. 

The R. temporaria transcriptome was published in 2015: 

A de novo Assembly of the Common Frog (Rana temporaria) Transcriptome and Comparison of Transcription Following Exposure to Ranavirus and Batrachochytrium dendrobatidis

Price *et al.* 2015, PLoS One


He sent me a link to the unannotated genome: 

https://dl.dropboxusercontent.com/u/13818137/Trinity.fasta

I'll use the same two samples to map to the transcriptome: 

Time: 210.343seconds
```
bwa index R.temp.trancriptome.fas
```
output: 
Rtk.Tcript.indexed.amb  Rtk.Tcript.indexed.ann  Rtk.Tcript.indexed.bwt  Rtk.Tcript.indexed.pac  Rtk.Tcript.indexed.sa



Align the demultiplexed individuals to the indexed genome. 
This is for one individual
~108min
```
  bwa aln R.temp.trancriptome.fas /gdc_home4/alexjvr/SE.CH.mapping/vora_01.fq.trim > Rt.tcript.vora01.sai

bwa aln R.temp.trancriptome.fas /gdc_home4/alexjvr/SE.CH.mapping/vora09.fq.trim > Rt.tcript.F21.sai
```


Convert to sam format
```
bwa samse R.temp.trancriptome.fas Rt.tcript.vora01.sai /gdc_home4/alexjvr/SE.CH.mapping/vora_01.fq.trim > Rt.tcript.vora01.sai.sam

bwa samse R.temp.trancriptome.fas Rt.tcript.F21.sai /gdc_home4/alexjvr/SE.CH.mapping/F21.fq.trim > Rt.tcript.F21.sai.sam
```

Now I have to use samtools

First, index the genome (this only needs to be done once) (~2min)
```
samtools faidx R.temp.trancriptome.fas
```

Next, convert SAM to BAM files. (A BAM file is just a binary version of a SAM files) (1min)
```
samtools import R.temp.trancriptome.fas.fai Rt.tcript.vora01.sai.sam Rt.tcript.vora01.sai.bam

samtools import R.temp.trancriptome.fas.fai Rt.tcript.F21.sai.sam Rt.tcript.F21.sai.bam
```

And then sort the BAM file (1min)
```
samtools sort Rt.tcript.vora01.sai.bam -o Rt.tcript.vora01.sai.bam.sorted

samtools sort Rt.tcript.F21.sai.bam -o Rt.tcript.F21.sai.bam.sorted

```

And last, we need Samtools to index the BAM file (10s)
```
samtools index Rt.tcript.vora01.sai.bam.sorted

samtools index Rt.tcript.F21.sai.bam.sorted
```

Now I need to check the output of the alignments. 
1. What proportion of the sequences mapped?
2. What's the comparison between samples?
3. 

For basic statistics use the flagstat command
```
samtools flagstat Rt.tcript.vora01.sai.bam.sorted
```

OR
the idxstats command
```
samtools idxstats Rt.tcript.vora01.sai.bam.sorted
```

###Results

I used the demultiplexed & trimmed reads from the following samples: 

SE data (F21)

```
samtools flagstat Rt.tcript.F21.sai.bam.sorted

2374365 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
343643 + 0 mapped (14.47% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```

And on one of the CH data (vora_01)

```
samtools flagstat Rt.tcript.vora01.sai.bam.sorted

3758034 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
503348 + 0 mapped (13.39% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```



###Mapping to the *Nanorana parkeri* genome

Sun *et al.* 2015, PNAS Published the genome of a from *Nanorana parkeri*. 

From a different family to *R.temp* (Dicroglossidae vs Ranidae), but seem to be fairly closely related: Pyron & Wiens 2011 MPE

According to the paper 

1. ~50% of the genome comprised TE

2. ~11Mb of amphibina specific conserved elements (when compared with Xenopus tropicalis)

3. Substantial blocks of homologous synteny with rare inter- or intrachromosomal rearrangements between Xenopus and Nanorana. 

Genome size ~2.3Gb

I will map my sequences to Nanorana as a test. 

NCBI Bioproject accession nr: PRJNA243398

```
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA_000935625.1_ASM93562v1/* .
```

I'm running this on GDCsrv1

** I'm not sure which file I need to use. A description can be found here: http://www.ncbi.nlm.nih.gov/genome/doc/ftpfaq/

But this has not clarified anything for me. 

I'm starting with the fasta file

Time to index: 
```
bwa index GCA_000935625.1_ASM93562v1_genomic.fna.gz
```

Align F21 and vora01 to the indexed genome
```
bwa aln GCA_000935625.1_ASM93562v1_genomic.fna.gz /gdc_home4/alexjvr/SE.CH.mapping/F21.fq.trim > Nan.F21.sai
```

... follow the protocol as above. 

Results: 

vora01
```
3758034 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
33603 + 0 mapped (0.89% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```

F21
```
2374365 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
17341 + 0 mapped (0.73% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```


##Transcriptome mapping


