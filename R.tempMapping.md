
#R.temp Mapping

##Rerun April 2016

I am retrying the mapping at UCLA with both Alan's genome and the newly available transcriptome.

I am using the SE dataset for the initial run.

###Mapping to Alan's R.temp genome

I am optimising the mapping and SNP calling protocol for R.temporaria using the draft genome from Alan Brelsford. The deNovo assembly takes >1 day per lane to run, and needs to be rerun as samples are added to the dataset.

SNP calling when mapping is possible using low coverage sequencing when using a probabilistic approch to SNP calling. So this is perfect for the Rana temporaria data.

Mapping quality

SNP calling

SNP filtering

Comparison with deNovo assembly

how much data is used/lost?

how many SNPs are called?

What is the overlap in SNPs between HiSeq runs

Time taken for SNP calling

Summary statistics

Pipeline

Overview of a pipeline is here:

http://www.ebi.ac.uk/training/sites/ebi.ac.uk.training/files/materials/2014/140217_AgriOmics/dan_bolser_snp_calling.pdf

**use bwa-mem

Align

Index genome

SNP calling

Filter SNPs


####1. Align to the genome

1hour on GDCsrv1
```
bwa index Rtk43.fa
```
output: 

Rtk43.fa.amb 

Rtk43.fa.ann 

Rtk43.fa.bwt 

Rtk43.fa.pac 

Rtk43.fa.sa

Align the demultiplexed individuals to the indexed genome. This is for one individual ~108min
```
bwa aln Rtk43.fa.gz /gdc_home4/alexjvr/SEFinalSamples/SEsamples_193.19/F21.fq.trim > Rt.F21.sai
```

To align all the individuals This is for all 451 individuals. Start 23:52
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

#for multiple files
for i in *.bam; do samtools sort -@10 "${i}" "${i}".sorted; done
```

And last, we need Samtools to index the BAM file (10s)

```
samtools index Rt.H2.bam.sorted.bam

for i in *.sorted.bam; do samtools index "${i}"; done
```
Now I need to check the output of the alignments. 1. What proportion of the sequences mapped? 2. What's the comparison between samples? 3.

For basic statistics use the flagstat command

```
samtools flagstat Rt.abnd03.sai.bam.sorted.bam
```
OR the idxstats command
```
samtools idxstats Rt.abnd03.sai.bam.sorted.bam
```
Results

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

####Mapping to the genome

#####4 November 2016

I realised that there's a chance that I will get the same number of reads from mapping as from de novo assembly, but that I could then have greater confidence in the mapped reads. As well as the added benefit of genome position. 

I will try to map 40 individuals to Alan's genome: 
These inividuals should describe the variation all across Switzerland. 

On fgcz47: /srv/kenlab/alexjvr_p1795/CHmapping

1. index the genome 

Few mins on fgcz47

```
bwa index Rtk43.fa
```
output: 

Rtk43.fa.amb 

Rtk43.fa.ann 

Rtk43.fa.bwt 

Rtk43.fa.pac 

Rtk43.fa.sa

2. Map multiple samples: 

script: 
```
for i; do
bwa mem -t 10 Rtk43.fa "${i}" > "${i}.sai"
done
```
-t meand running 10 CPU. This seems to be quite fast: 

Real time: 311.058 sec; CPU: 3032.208 sec  per sample

```
find -type f -name "*fq.trim.gz" -exec ./script {} +
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
Now I need to check the output of the alignments. 1. What proportion of the sequences mapped? 2. What's the comparison between samples? 3.

For basic statistics use the flagstat command

```
samtools flagstat Rt.abnd03.sai.bam.sorted.bam
```
OR the idxstats command
```
samtools idxstats Rt.abnd03.sai.bam.sorted.bam
```
Results




#8Feb 2017: Map all samples to the genome

Based on the results before, I found that mapping was really improved when using bwa_mem. In the end I will end up with pretty much the 
same number of loci called in the dataset compared with de Novo assembly, but I should have a lot more confidence in the call. And there 
should be less problems with the alignments. 

I'm transferring all the samples from Beelzebufo onto the fgcz server for mapping: 

Location: Beelzebufo/CHcomplete/demultiplexed_trim_H/samples.trim_CH1027/

Size: 221Gb

rsync started: 8Feb 10:21
```
rsync -r samples.trim_CH1027 fgcz47:/srv/kenlab/alexjvr_p1795/CHall/

```

Map a batch of 260 individuals
```
for i in $(ls /srv/kenlab/alexjvr_p1795/CHall/samples.trim_CH1027/batch1/*); do bwa mem -t 5 Rtk43.fa "${i}" > "${i}.sai"; done
```
Started 9 Feb 7:26 (expected end = Sat 7:00)

convert to .bam and delete all the sam files
```
for i in $(ls *sai); do samtools import Rtk43.fa.fai "${i}" "${i}".bam; done

```

sort and index. 
```
for i in $(ls *bam); do samtools sort -@10 "${i}" "${i}".sorted; done

for i in $(ls *bam); do samtools index "${i}"; done
```

This generates *.bam and *.bam.bai files. The .bai file is a companion file to the .bam. 


When I check the mapping, I get extremely high mapping rates: 

```
samtools flagstat zeni_11.fq.trim.gz.sai.bam.sorted.bam

5349830 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
779367 + 0 supplementary
0 + 0 duplicates
5021655 + 0 mapped (93.87%:-nan%)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (-nan%:-nan%)
0 + 0 with itself and mate mapped
0 + 0 singletons (-nan%:-nan%)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)

```

This decreases to about 40% mapping rate when I filter for mapping quality of 30. Christine doesn't do any filtering on the bam files before running freebayes. (Only mark-duplicates, which I can't do with the ddRAD data). 

So I will run freebayes as is on the unfiltered bam files.



####Freebayes

Located on the fgcz47 server: /usr/local/ngseq/src/freebayes/

Tutorials from Erik
https://github.com/ekg/alignment-and-variant-calling-tutorial

http://clavius.bc.edu/~erik/CSHL-advanced-sequencing/freebayes-tutorial.html


And from the BAG2016 workshop: /Users/alexjvr/2016RADAnalysis/bag2016/tuesday

For input files, make sure the bam files are sorted and indexed. To check whether the file is sorted: 
SO:coordinate shows that its sorted. 
```
samtools view -H abnd_01.fq.trim.gz.sai.bam.sorted.bam |head

@HD	VN:1.3	SO:coordinate
@SQ	SN:scaffold1.1|size15254	LN:15254
@SQ	SN:scaffold2.1|size11579	LN:11579
@SQ	SN:scaffold3.1|size10991	LN:10991
@SQ	SN:scaffold4.1|size12417	LN:12417
@SQ	SN:scaffold5.1|size10895	LN:10895
@SQ	SN:scaffold6.1|size10551	LN:10551
@SQ	SN:scaffold7.1|size9739	LN:9739
@SQ	SN:scaffold8.1|size15096	LN:15096
@SQ	SN:scaffold9.1|size15729	LN:15729

```

Rename all the files
```
for i in *.fq.trim.gz.sai.bam.sorted.bam; do mv -v -- "$i" "${i%.fq.trim.gz.sai.bam.sorted.bam}.bam"; done

for i in *.fq.trim.gz.sai.bam.sorted.bam.bai; do mv -v -- "$i" "${i%.fq.trim.gz.sai.bam.sorted.bam.bai}.bai"; done
```

Add reading group information to all the samples. This can be done during the bwa mem step already, but I didn't realise that while I was mapping. However, there's a Picard tool with which to replace or add new reading group information. 
```
bwa mem -aM -R "@RG\tID:Seq01\tSM:Seq01\tPL:ILLUMINA\tPI:330" Refgen Seq01.fastq > Seq01.sam 
```

Importantly, the RG ID needs to be unique for Freebayes to work. The sample name (SM) doesn't matter. 

Picard's AddOrRemoveReadGroups.jar is a useful tool for editing the RG. 
```
java -jar /usr/local/ngseq/src/picard-tools-2.6/picard.jar AddOrReplaceReadGroups I=zeni_02.fq.trim.gz.sai.bam.sorted.bam O=zeni_02.bam RGID=zeni.02 RGLB=lib1 RGPL=illumina RGPU=unit1 RGSM=zeni.02
```

Add RG to all files: 
```
for i in *.bam; do java -jar /usr/local/ngseq/src/picard-tools-2.6/picard.jar AddOrReplaceReadGroups I=${i} O=${i}.RG RGID=${i} RGLB=lib1 RGPL=illumina RGPU=unit1 RGSM=${i}; done
```


To run many bam files, first create a file with all the .bam names
```
ls *sorted.bam > bam.files.txt
```


And run this with freebayes on fgcz4:
```
/usr/local/ngseq/src/freebayes/bin/freebayes -f Rtk43.fa --no-complex --use-best-n-alleles 4 --min-base-quality 3 --min-mapping-quality 20 --no-population-priors --hwe-priors-off -dd -L zeni.bamfiles.txt  > zeni.vcf 2>&1 |tail -10000 |tee failure.txt
```

-dd: generates information for debugging
2>&1 |tail -10000 |tee failure.txt  : writes the failure to a text file. 














###Mapping to the transcriptome.

The R. temporaria transcriptome was published in 2015:

A de novo Assembly of the Common Frog (Rana temporaria) Transcriptome and Comparison of Transcription Following Exposure to Ranavirus and Batrachochytrium dendrobatidis

Price et al. 2015, PLoS One

He sent me a link to the unannotated genome:

https://dl.dropboxusercontent.com/u/13818137/Trinity.fasta

I'll use the same two samples to map to the transcriptome:

Time: 210.343seconds

```
bwa index R.temp.trancriptome.fas
```

output: 

Rtk.Tcript.indexed.amb 

Rtk.Tcript.indexed.ann 

Rtk.Tcript.indexed.bwt

Rtk.Tcript.indexed.pac 

Rtk.Tcript.indexed.sa

Align the demultiplexed individuals to the indexed genome. This is for one individual ~108min

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

Now I need to check the output of the alignments. 1. What proportion of the sequences mapped? 2. What's the comparison between samples? 3.

For basic statistics use the flagstat command

```
samtools flagstat Rt.tcript.vora01.sai.bam.sorted
```
OR the idxstats command
```
samtools idxstats Rt.tcript.vora01.sai.bam.sorted
```
Results

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

####Mapping to the Nanorana parkeri genome

Sun et al. 2015, PNAS Published the genome of a from Nanorana parkeri.

From a different family to R.temp (Dicroglossidae vs Ranidae), but seem to be fairly closely related: Pyron & Wiens 2011 MPE

According to the paper

~50% of the genome comprised TE

~11Mb of amphibina specific conserved elements (when compared with Xenopus tropicalis)

Substantial blocks of homologous synteny with rare inter- or intrachromosomal rearrangements between Xenopus and Nanorana.

Genome size ~2.3Gb

I will map my sequences to Nanorana as a test.

NCBI Bioproject accession nr: PRJNA243398

wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA_000935625.1_ASM93562v1/* .
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
