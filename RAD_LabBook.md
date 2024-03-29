##Index

1. Demultiplexing, Quality filter, Adapter removal
2. Pyrad tests
3. Mapping tests1. Demultiplexing, Quality filter, Adapter removal

*Monday 23March 2014*

I'm losing a lot of reads with process_radtags, so I will try a different method of barcode splitting and filtering. Christine Grossen suggests splitting barcodes first, and less stringent filtering before running the SNPcalling pipeline. I will install fastx-barcode splitter.

Install FASTX Toolkit

-fastx_toolkit-0.0.14 from http://hannonlab.cshl.edu/fastx_toolkit/download.html
-check that libgtextutils-0.7 is installed (or latest version)

```
./configure && make && make install
```

I get an error

```
configure: error: The pkg-config script could not be found or is too old.  Make sure it
is in your PATH or set the PKG_CONFIG environment variable to the full
path to pkg-config.

Alternatively, you may set the environment variables GTEXTUTILS_CFLAGS
and GTEXTUTILS_LIBS to avoid the need to call pkg-config.
See the pkg-config man page for more details.
```

- Download latest version from here: http://pkgconfig.freedesktop.org/releases/
- There is some catch22 with pkgconfig dependant on glib, and vice versa.. 


Check on the server to find FASTX: 
```
/usr/local/ngseq/stow/fastx-toolkit_0.0.13s
```

*1: Uncompress tarball*

To uncompress them, execute the following command(s) depending on the extension:
```
$ tar zxf file.tar.gz
$ tar zxf file.tgz
$ tar jxf file.tar.bz2
$ tar jxf file.tbz2
```


###Barcode splitting with FASTX toolkit

test for H1 

1. Decompress files
```
pigz -dc 20150225.A-H1_R1.fastq.gz > H1.fastq
```

2. send to fast_barcode_splitter
```
cat H1.fastq | /usr/local/ngseq/stow/fastx_toolkit-0.0.13/bin/fastx_barcode_splitter.pl --bcfile barcodes_fastx_H1 --prefix H1_fastx --bol --mismatches 1
```

*process_radtags*

-what could the problem be with the process with process_radtags? 
-most of the loss (avg = 26%) is from “no ragtag”. Which I think means no restriction site.. 

-> reprocess H1 with process_radtags, but remove the enzyme check and include the cut site in the barcode. (If your enzyme is not yet supported by Stacks use the --disable_rad_check option)

```
/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150225_RUN171/20150225.A-H1_R1.fastq.gz -o ./H1 -y fastq -b ~/barcodes -e ecoRI -r -c -q
```


Compared with Simone’s example: 

##insert table here..

but don’t specify an enzyme, and use the modified barcode file:
```
mkdir H1 
/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150225_RUN171/20150225.A-H1_R1.fastq.gz -o ./H1 -y fastq -b ~/barcodes_RE --disable_rad_check -r -c -q -D
```
this takes a few mins to run (>30min per lane)


So there is very little differences between these two options of processing. Both remove ~35%, of which 20% is PhiX. So around 15% filtered based on barcodes.. 

-> repeat allowing 2 mismatches

```
mkdir H1.2 
/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150225_RUN171/20150225.A-H1_R1.fastq.gz -o ./H1.2 -y fastq -b ~/barcodes_RE --disable_rad_check -r -c -q -D —barcode_dist_1 2
```


So, a 6% gain in sequences. But the % of sequences filtered for quality has increased. Why? 

To check whether the 2mm might be resulting in an excess loss or gain between barcodes, I plot a regression between the read counts of 1mm vs 2mm. 


###insert plot here












And looked at the table of sequence counts: if some samples are losing sequences, while others gain, then these sequences are likely incorrectly assigned to the new barcode. This is indeed the case with my data: 

##insert table here


*Optimal mm: 1*



###Secure copy files onto and from the server
```
scp -r alexjvr@gdcsrv1.ethz.ch:popgts_c94d6_20150525/* .
```

##Filter for Adapters

Christine recommends AdapterRemoval. This is her example code

```
AdapterRemoval --file1 <(zcat $wGBS/$R1) --file2 <(zcat $wGBS/$R2)  --trimns --trimqualities --minquality 20 --collapse --minlength 50 --qualitybase 33 --pcr1 AGATCGGAAGAGCGGTTCAGCAGGAATGCCGAG --pcr2 AATGATACGGCGACCACCGAGATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT --basename $COLL/$trunk >& $wGBS/logfiles/AdRem_$trunk.log
```

set up barcode and raw data file on the mac: 
```
/NGSpipelines/Filter1
```

Run process_radtags (only -r: rescue radtags. -c -q filters removed, since data will be cleaned downstream): 
```
process_radtags -i fastq -f 20141219.B-p276_h2_R1.fastq -o . -y fastq -b barcode_H2 -e ecoRI -r

215305263 total sequences;
  17389044 ambiguous barcode drops;
  0 low quality read drops;
  30108440 ambiguous RAD-Tag drops;
167807779 retained reads.
```

This means a loss of only ~3% of the reads. 

##3. Run AdapterRemoval on demultiplexed data
```
adapterremoval --file1 csee_02.fq --basename AdRem_$.trunk --trimns --trimqualities --minquality 20 --adapter1 AATGATACGGCGACCACCGAGATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT
```

Some of the removed sequences were part of the AATT EcoRI cutsite. So rerun the demultiplexing and remove the cutsite: 

Redo this with the disable_rad_check option selected (barcode file should be updated to include the cutsite) 

```
GCATGAATT       shwe_09
AACCAAATT       shwe_11
CGATCAATT       csee_02
TCGATAATT       csee_03
TGCATAATT       csee_04
CAACCAATT       csee_05
GGTTGAATT       csee_06
AAGGAAATT       csee_07
AGCTAAATT       csee_08
ACACAAATT       csee_09
AATTAAATT       csee_10
ACGGTAATT       csee_11
ACTGGAATT       csee_12
ACTTCAATT       csee_13
```
```
process_radtags -i fastq -f 20141219.B-p276_h2_R1.fastq -o . -y fastq -b barcode_H2 --disable_rad_check -r -D

215305263 total sequences;
  47501389 ambiguous barcode drops;
  0 low quality read drops;
  0 ambiguous RAD-Tag drops;
167803874 retained reads.
```

20% PhiX
~2.5% lost sequences

```
adapterremoval --file1 csee_02.fq --basename AdRem --trimns --trimqualities --minquality 20 --adapter1 AATGATACGGCGACCACCGAGATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT
```

And batch script: 
```
for x in *.fq
do adapterremoval --file1 ${x} --basename AdRem_${x} --trimns --trimqualities --minquality 20 --adapter1 AATGATACGGCGACCACCGAGATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT
done
```

##PyRAD tests     
TEST pyRAD with SE ddRAD test data on server 

Tutorial from:
 http://nbviewer.ipython.org/gist/dereneaton/dc6241083c912519064e/tutorial_ddRAD_3.0.4.ipynb

```
%%bash
## download the data
wget -q http://www.dereneaton.com/downloads/simddrads.zip
## unzip the data
unzip simddrads.zip

%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -n

	#new params.txt file created

%%bash
nano params.txt
```

-> change the path to vsearch: /usr/local/ngseq/src/vsearch-1.1.3/vsearch
->change the path to muscle: /usr/local/ngseq/src/muscle-3.8.31/muscle
```
%%bash
sed -i '/## 4. /c\/usr/local/ngseq/src/vsearch-1.1.3/vsearch    ## 4. vsearch... ' ./params.txt
sed -i '/## 5. /c\/usr/local/ngseq/src/muscle-3.8.31/muscle    ## 5. muscle... ' ./params.txt
```

->Change the other parameters:
```
%%bash
sed -i '/## 6. /c\TGCAG,AATT                 ## 6. cutsites... ' ./params.txt
sed -i '/## 10. /c\.85                       ## 10. lowered clust thresh' ./params.txt
sed -i '/## 11. /c\ddrad                     ## 11. datatype... ' ./params.txt
sed -i '/## 14. /c\c85d6m4p3                 ## 14. prefix name ' ./params.txt
sed -i '/## 19./c\0                     ## 19. errbarcode... ' ./params.txt
sed -i '/## 24./c\10                    ## 24. maxH... ' ./params.txt
sed -i '/## 30./c\*                     ## 30. outformats... ' ./params.txt
```

###Demultiplexing step
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 1
```

Follow the tutorial… 

*2. Quality filtering* 
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 2
```

*3. Within-sample clustering*
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 3
```

*Steps 4 & 5: Consensus base calling*
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 4

%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 5
```

*Step 6: Across-sample clustering*
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 6
```

*Step 7: Statistics and output files*
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 7
```



###H1 pyRAD with SE ddRAD test run on server 

Tutorial from http://nbviewer.ipython.org/gist/dereneaton/dc6241083c912519064e/tutorial_ddRAD_3.0.4.ipynb

symlink to my data in /srv/kenlab/alexjvr_p1795/pyrad$
```
ln -s /srv/gstore4users/p1795/HiSeq_20150225_RUN171/20150225.A-H1_R1.fastq.gz
```

update barcode file for the run (insert sample names): 
```
oalp01  GCATG
oalp02  AACCA
oalp03  CGATC
oalp04  TCGAT
oalp05  TGCAT
oalp06  CAACC
oalp07  GGTTG
oalp08  AAGGA
oalp09  AGCTA
oalp10  ACACA
tals01  AATTA
```

create new params.txt file
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -n
```

change the path to vsearch: /usr/local/ngseq/src/vsearch-1.1.3/vsearch
change the path to muscle: /usr/local/ngseq/src/muscle-3.8.31/muscle
```
%%bash
nano params.txt
```
 OR 
```
%%bash
sed -i '/## 4. /c\/usr/local/ngseq/src/vsearch-1.1.3/vsearch    ## 4. vsearch... ' ./params.txt
sed -i '/## 5. /c\/usr/local/ngseq/src/muscle-3.8.31/muscle    ## 5. muscle... ' ./params.txt
```

Change the other parameters:
```
%%bash

sed -i '/## 1. /c\./	                ## 1. working directory ' ./params.txt
sed -i '/## 2. /c\/srv/gstore4users/p1795/HiSeq_20150225_RUN171/20150225.A-H1_R1.fastq.gz                ## 2. Raw data H1 ' ./params.txt
sed -i '/## 3. /c\barcodesH1                 ## 3. barcodes ' ./params.txt
sed -i '/## 6. /c\AATT,TAA                 ## 6. cutsites EcoRI/MseI ' ./params.txt
sed -i '/## 7. /c\8                 ## 7. Nprocessors ' ./params.txt
sed -i '/## 8. /c\10                 ## 8. mindepth for base calls ' ./params.txt
sed -i '/## 9. /c\4                 ## 9. Max N ' ./params.txt
sed -i '/## 10. /c\.94                       ## 10. clust thresh ~5bpmm' ./params.txt
sed -i '/## 11. /c\ddrad                     ## 11. datatype... ' ./params.txt
sed -i '/## 12. /c\45                     ## 12. high mincov ' ./params.txt

sed -i '/## 13. /c\4                     ## 13.maxsharedH  ' ./params.txt
sed -i '/## 14. /c\H1_d10n4c94mc45h4_                 ## 14. prefix name ' ./params.txt
sed -i '/## 21./c\10                    ## 21. check for adapter... ' ./params.txt
sed -i '/## 24./c\10                    ## 24. maxH... ' ./params.txt
sed -i '/## 30./c\*                     ## 30. outformats... ' ./params.txt
sed -i '/## 34./c\2                     ## 34. excludesingletons... ' ./params.txt
```

Demultiplexing step
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 1
```
OR 

demultiplex with process radtags and refer to demultiplexed files in params.txt

```
sed -i '/## 18. /c\/srv/kenlab/alexjvr_p1795/Seqcount_processRadtags_20150316/H1.1/*.fq                     ## 18.demultiplexed H1(1mm)  ' ./params.txt
```

Follow the tutorial… 

```
#2. Quality filtering. 
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 2

#3. Within-sample clustering (~2hrs/lane)
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 3

#Steps 4 & 5: Consensus base calling (step 4 <9hrs, 5 <1.5hrs)
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 4

%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 5

#Step 6: Across-sample clustering

%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 6

#Step 7: Statistics and output files

%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 7
```

###H1.2 - less stringent filtering
```
sed -i '/## 1. /c\./	                ## 1. working directory ' ./params.txt
sed -i '/## 2. /c\/srv/gstore4users/p1795/HiSeq_20150225_RUN171/20150225.A-H1_R1.fastq.gz                ## 2. Raw data H1 ' ./params.txt
sed -i '/## 3. /c\barcodesH1                 ## 3. barcodes ' ./params.txt
sed -i '/## 6. /c\AATT,TAA                 ## 6. cutsites EcoRI/MseI ' ./params.txt
sed -i '/## 7. /c\8                 ## 7. Nprocessors ' ./params.txt
sed -i '/## 8. /c\5                 ## 8. mindepth for base calls ' ./params.txt
sed -i '/## 9. /c\8                 ## 9. Max N ' ./params.txt
sed -i '/## 10. /c\.90                       ## 10. clust thresh ~5bpmm' ./params.txt
sed -i '/## 11. /c\ddrad                     ## 11. datatype... ' ./params.txt
sed -i '/## 12. /c\40                     ## 12. high mincov ' ./params.txt

sed -i '/## 13. /c\4                     ## 13.maxsharedH  ' ./params.txt
sed -i '/## 14. /c\H1.2                 ## 14. prefix name ' ./params.txt
sed -i '/## 21./c\10                    ## 21. check for adapter... ' ./params.txt
sed -i '/## 24./c\10                    ## 24. maxH... ' ./params.txt
sed -i '/## 30./c\*                     ## 30. outformats... ' ./params.txt
sed -i '/## 32./c\50                    ## 32. keeptrimmed >50... ' ./params.txt
sed -i '/## 34./c\2                     ## 34. excludesingletons... ' ./params.txt
```

*12 April 2015*
###H1.3 (started 18:00)
```
==** parameter inputs for pyRAD version 3.0.4  **======================== affected step ==
./                ## 1. working directory 
                  ## 2. raw data
                 ## 3. barcodes 
/usr/local/ngseq/src/vsearch-1.1.3/vsearch    ## 4. vsearch... 
/usr/local/ngseq/src/muscle-3.8.31/muscle    ## 5. muscle... 
AATT,TAA                 ## 6. cutsites EcoRI/MseI 
8                 ## 7. Nprocessors 
5                 ## 8. mindepth for base calls 
8                 ## 9. Max N 
.80                       ## 10. clust thresh
ddrad                     ## 11. datatype... 
20                     ## 12. high mincov 
4                     ## 13.maxsharedH  
H1.3                 ## 14. prefix name 
==== optional params below this line ===================================  affected step ==
                       ## 15.opt.: select subset (prefix* only selector)            (s2-s7)
                       ## 16.opt.: add-on (outgroup) taxa (list or prefix*)         (s6,s7)
                       ## 17.opt.: exclude taxa (list or prefix*)                   (s7)
/srv/kenlab/alexjvr_p1795/Seqcount_processRadtags_20150316/H1.1/*.fq                       ## 18.opt.: loc. of de-multiplexed data                      (s2)
                       ## 19.opt.: maxM: N mismatches in barcodes (def= 1)          (s1)
                       ## 20.opt.: phred Qscore offset (def= 33)                    (s2)
10                    ## 21. check for adapter... 
                       ## 22.opt.: a priori E,H (def= 0.001,0.01, if not estimated) (s5)
                       ## 23.opt.: maxN: max Ns in a cons seq (def=5)               (s5)
10                    ## 24. maxH... 
                       ## 25.opt.: ploidy: max alleles in cons seq (def=2;see docs) (s4,s5)
                       ## 26.opt.: maxSNPs: (def=100). Paired (def=100,100)         (s7)
                       ## 27.opt.: maxIndels: within-clust,across-clust (def. 3,99) (s3,s7)
                       ## 28.opt.: random number seed (def. 112233)              (s3,s6,s7)
                       ## 29.opt.: trim overhang left,right on final loci, def(0,0) (s7)
*                     ## 30. outformats... 
                       ## 31.opt.: maj. base call at depth>x<mindepth (def.x=mindepth) (s5)
50                    ## 32. keeptrimmed >50... 
                       ## 33.opt.: max stack size (int), def= max(500,mean+2*SD)    (s3)
2                     ## 34. excludesingletons... 
                       ## 35.opt.: use hierarchical clustering (def.=0, 1=yes)      (s6)
                       ## 36.opt.: repeat masking (def.=1='dust' method, 0=no)      (s3,s6)
                       ## 37.opt.: vsearch max threads per job (def.=6; see docs)   (s3,s6)
==== optional: list group/clade assignments below this line (see docs) ==================
```

Slightly more SNPs, but still much lower than expected: 5363 unlinked SNPs

— Christine suggests that I’m over-clustering my data, so I should be less stringent than this!
— Remember to plot heterozygosity to test this. 


##Within population pyrad

I ran pyrad on shwe and wdji independently
```
==** parameter inputs for pyRAD version 3.0.4  **======================== affected step ==
./                ## 1. working directory 
                  ## 2. raw data
                 ## 3. barcodes 
/usr/local/ngseq/src/vsearch-1.1.3/vsearch    ## 4. vsearch... 
/usr/local/ngseq/src/muscle-3.8.31/muscle    ## 5. muscle... 
AATT,TAA                 ## 6. cutsites EcoRI/MseI 
8                 ## 7. Nprocessors 
6                 ## 8. mindepth for base calls 
8                 ## 9. Max N 
.90                       ## 10. clust thresh
ddrad                     ## 11. datatype... 
5                     ## 12. high mincov 
4                     ## 13.maxsharedH  
wdji                 ## 14. prefix name 
==== optional params below this line ===================================  affected step ==
                       ## 15.opt.: select subset (prefix* only selector)            (s2-s7)
                       ## 16.opt.: add-on (outgroup) taxa (list or prefix*)         (s6,s7)
                       ## 17.opt.: exclude taxa (list or prefix*)                   (s7)
@/srv/kenlab/alexjvr_p1795/Seqcount_processRadtags_20150316/H1.1/wdji/*.fq                       ## 18.opt.: loc. of de-multiplexed data                      (s2)
                       ## 19.opt.: maxM: N mismatches in barcodes (def= 1)          (s1)
                       ## 20.opt.: phred Qscore offset (def= 33)                    (s2)
0                    ## 21. check for adapter... 
                       ## 22.opt.: a priori E,H (def= 0.001,0.01, if not estimated) (s5)
                       ## 23.opt.: maxN: max Ns in a cons seq (def=5)               (s5)
10                    ## 24. maxH... 
                       ## 25.opt.: ploidy: max alleles in cons seq (def=2;see docs) (s4,s5)
                       ## 26.opt.: maxSNPs: (def=100). Paired (def=100,100)         (s7)
                       ## 27.opt.: maxIndels: within-clust,across-clust (def. 3,99) (s3,s7)
                       ## 28.opt.: random number seed (def. 112233)              (s3,s6,s7)
                       ## 29.opt.: trim overhang left,right on final loci, def(0,0) (s7)
*                     ## 30. outformats... 
                       ## 31.opt.: maj. base call at depth>x<mindepth (def.x=mindepth) (s5)
50                    ## 32. keeptrimmed >50... 
                       ## 33.opt.: max stack size (int), def= max(500,mean+2*SD)    (s3)
2                     ## 34. excludesingletons... 
                       ## 35.opt.: use hierarchical clustering (def.=0, 1=yes)      (s6)
                       ## 36.opt.: repeat masking (def.=1='dust' method, 0=no)      (s3,s6)
                       ## 37.opt.: vsearch max threads per job (def.=6; see docs)   (s3,s6)

```

Final SNPs for 
*wdji*: 

total var= 32712
total pis= 19752
sampled unlinked SNPs= 8672
sampled unlinked bi-allelic SNPs= -8022

taxa    total   dpt.me  dpt.sd  d>5.tot d>5.me  d>5.sd  badpairs
wdji_03 148316  6.909   39.497  36066   19.879  78.677  0
wdji_04 100824  6.337   34.547  22007   19.382  72.436  0
wdji_05 123288  6.901   35.171  29638   20.072  70.101  0
wdji_06 116497  6.721   44.796  27087   19.91   91.654  0
wdji_07 66920   5.075   22.118  11217   17.357  52.279  0
wdji_08 87284   6.187   28.133  19044   18.733  58.506  0
wdji_09 97701   6.537   42.04   21948   19.774  87.396  0
wdji_10 122923  6.714   40.16   28446   19.987  82.08   0
wdji_11 105577  6.251   34.371  22655   19.313  72.696  0
wdji_14 84905   5.749   41.297  16106   18.997  93.649  0


*stls*: 

total var= 35813
total pis= 21241
sampled unlinked SNPs= 9348
sampled unlinked bi-allelic SNPs= -8716

taxa    total   dpt.me  dpt.sd  d>5.tot d>5.me  d>5.sd  badpairs
stls_03 115317  6.902   31.016  28128   19.825  60.993  0
stls_05 109322  6.365   33.875  24731   18.877  69.764  0
stls_08 78727   5.57    29.761  14597   18.419  67.606  0
stls_09 95944   5.855   29.936  19087   18.731  65.529  0
stls_11 103218  6.391   33.547  22682   19.494  69.986  0
stls_12 98686   6.428   34.561  21892   19.488  71.848  0
stls_13 101822  6.572   45.184  23149   19.7    93.561  0
stls_14 137135  6.826   37.221  32799   19.824  74.615  0
stls_15 139686  7.195   40.105  35428   20.268  78.164  0
stls_16 100956  6.327   30.649  22385   19.051  63.444  0



###H1.5

I will rerun H01 samples together, with the same params.txt file as in H1.4, but with less samples for the minimum clustering -> decreased from 20 to 8. 
Note. I am still excluding singletons in my runs. I will test the hierarchical search next.
```
==** parameter inputs for pyRAD version 3.0.4  **======================== affected step ==
./                ## 1. working directory 
                  ## 2. raw data
                 ## 3. barcodes 
/usr/local/ngseq/src/vsearch-1.1.3/vsearch    ## 4. vsearch... 
/usr/local/ngseq/src/muscle-3.8.31/muscle    ## 5. muscle... 
AATT,TAA                 ## 6. cutsites EcoRI/MseI 
8                 ## 7. Nprocessors 
5                 ## 8. mindepth for base calls 
8                 ## 9. Max N 
.80                       ## 10. clust thresh ~5bpmm
ddrad                     ## 11. datatype... 
8                     ## 12. high mincov 
4                     ## 13.maxsharedH  
H1.5                 ## 14. prefix name 
==== optional params below this line ===================================  affected step ==
                       ## 15.opt.: select subset (prefix* only selector)            (s2-s7)
                       ## 16.opt.: add-on (outgroup) taxa (list or prefix*)         (s6,s7)
                       ## 17.opt.: exclude taxa (list or prefix*)                   (s7)
@/srv/kenlab/alexjvr_p1795/Seqcount_processRadtags_20150316/H1.1/*.fq                       ## 18.opt.: loc. of de-multiplexed data                      (s2)
                       ## 19.opt.: maxM: N mismatches in barcodes (def= 1)          (s1)
                       ## 20.opt.: phred Qscore offset (def= 33)                    (s2)
10                    ## 21. check for adapter... 
                       ## 22.opt.: a priori E,H (def= 0.001,0.01, if not estimated) (s5)
                       ## 23.opt.: maxN: max Ns in a cons seq (def=5)               (s5)
10                    ## 24. maxH... 
                       ## 25.opt.: ploidy: max alleles in cons seq (def=2;see docs) (s4,s5)
                       ## 26.opt.: maxSNPs: (def=100). Paired (def=100,100)         (s7)
                       ## 27.opt.: maxIndels: within-clust,across-clust (def. 3,99) (s3,s7)
                       ## 28.opt.: random number seed (def. 112233)              (s3,s6,s7)
                       ## 29.opt.: trim overhang left,right on final loci, def(0,0) (s7)
*                     ## 30. outformats... 
                       ## 31.opt.: maj. base call at depth>x<mindepth (def.x=mindepth) (s5)
50                    ## 32. keeptrimmed >50... 
                       ## 33.opt.: max stack size (int), def= max(500,mean+2*SD)    (s3)
2                     ## 34. excludesingletons... 
                       ## 35.opt.: use hierarchical clustering (def.=0, 1=yes)      (s6)
                       ## 36.opt.: repeat masking (def.=1='dust' method, 0=no)      (s3,s6)
                       ## 37.opt.: vsearch max threads per job (def.=6; see docs)   (s3,s6)


33224       ## loci with > minsp containing data
24971       ## loci with > minsp containing data & paralogs removed
24971       ## loci with > minsp containing data & paralogs removed & final filtering

total var= 80686
total pis= 46872
sampled unlinked SNPs= 15766
sampled unlinked bi-allelic SNPs= -56114

Avg depth 5.3-8.6x
```

###H1.6
Decrease the mincov, and include singletons, to get an idea of all the loci at different levels of coverage. 
```
.80                       ## 10. clust thresh ~5bpmm
2                     ## 12. high mincov 
                     ## 34. include singletons... 

Results:

total var= 348706
total pis= 124847
sampled unlinked SNPs= 79092
sampled unlinked bi-allelic SNPs= -139816

Avg depth 
```

##Formal pyrad tests

*Aim*:

To test optimal clustering conditions (85 - 99%)
To test sensitivity do depth (6-10)

Tests will be set up with the following parameters. First I will vary the coverage (6, 8, & 10mindepth), then the clustering. 

ETHZ servers: pyrad v. 3.0.3 - max 20 cores
FGCZ servers: pyrad v. 3.0.4 - max 8 cores
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 3
```
==** parameter inputs for pyRAD version 3.0.3  **======================== affected step ==
./                          ## 1. Working directory                                 (all)
               ## 2. Loc. of non-demultiplexed files (if not line 16)  (s1)
               ## 3. Loc. of barcode file (if not line 16)             (s1)
vsearch                     ## 4. command (or path) to call vsearch (or usearch)    (s3,s6)
muscle                      ## 5. command (or path) to call muscle                  (s3,s7)
TGCAG                       ## 6. Restriction overhang (e.g., C|TGCAG -> TGCAG)     (s1,s2)
8/20                           ## 7. N processors (parallel)                           (all)
6-10                           ## 8. Mindepth: min coverage for a cluster              (s4,s5)
4                           ## 9. NQual: max # sites with qual < 20 (or see line 20)(s2)
.88                         ## 10. Wclust: clustering threshold as a decimal        (s3,s6)
ddrad                         ## 11. Datatype: rad,gbs,ddrad,pairgbs,pairddrad        (all)
14                           ## 12. MinCov: min samples in a final locus             (s7)
3                           ## 13. MaxSH: max inds with shared hetero site          (s7)
c88d6m4p3                   ## 14. Prefix name for final output (no spaces)         (s7)
==== optional params below this line ===================================  affected step ==
                       ## 15.opt.: select subset (prefix* only selector)            (s2-s7)
                       ## 16.opt.: add-on (outgroup) taxa (list or prefix*)         (s6,s7)
                       ## 17.opt.: exclude taxa (list or prefix*)                   (s7)
./*fq.gz                       ## 18.opt.: loc. of de-multiplexed data                      (s2)
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
                       ## 30.opt.: output formats: p,n,a,s,v,u,t,m,k,g,* (see docs) (s7)
                       ## 31.opt.: maj. base call at depth>x<mindepth (def.x=mindepth) (s5)
50                       ## 32.opt.: keep trimmed reads (def=0). Enter min length.    (s2)
                       ## 33.opt.: max stack size (int), def= max(500,mean+2*SD)    (s3)
2                       ## 34.opt.: minDerep: exclude dereps with <= N copies, def=1 (s3)
                       ## 35.opt.: use hierarchical clustering (def.=0, 1=yes)      (s6)
```



I will use 2 samples each from the following 14 populations: 

##insert map here



C88d6  - gdc (20cores) started: 11:03 19April2015

C88d8  - fgcz (8cores) started: 17:05 19April2015

C88d10 - fgcz (8cores) started: 10:50 19April2015 - 17:00 19April2015


*23 April 2015*

Rerun wdji at c94d6 (on fgcz)

location of demultiplexed wdji samples
```
/srv/kenlab/alexjvr_p1795/Seqcount_processRadtags_20150316/H1.1
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 3
```

####Start the first big pyRAD run to see how long it will take: 

*23 April 2015 @ 19:23*


All demultiplexed samples from H1-9
```
==** parameter inputs for pyRAD version 3.0.3  **======================== affected step ==
./                          ## 1. Working directory                                 (all)
                ## 2. Loc. of non-demultiplexed files (if not line 16)  (s1)
                ## 3. Loc. of barcode file (if not line 16)             (s1)
vsearch                     ## 4. command (or path) to call vsearch (or usearch)    (s3,s6)
muscle                      ## 5. command (or path) to call muscle                  (s3,s7)
TGCAG                       ## 6. Restriction overhang (e.g., C|TGCAG -> TGCAG)     (s1,s2)
20                           ## 7. N processors (parallel)                           (all)
6                           ## 8. Mindepth: min coverage for a cluster              (s4,s5)
4                           ## 9. NQual: max # sites with qual < 20 (or see line 20)(s2)
.94                         ## 10. Wclust: clustering threshold as a decimal        (s3,s6)
ddrad                         ## 11. Datatype: rad,gbs,ddrad,pairgbs,pairddrad        (all)
100                           ## 12. MinCov: min samples in a final locus             (s7)
                           ## 13. MaxSH: max inds with shared hetero site          (s7)
H1-9_c94d6m4                   ## 14. Prefix name for final output (no spaces)         (s7)
==== optional params below this line ===================================  affected step ==
                       ## 15.opt.: select subset (prefix* only selector)            (s2-s7)
                       ## 16.opt.: add-on (outgroup) taxa (list or prefix*)         (s6,s7)
                       ## 17.opt.: exclude taxa (list or prefix*)                   (s7)
~/demultiplexed/*fq.gz                       ## 18.opt.: loc. of de-multiplexed data                      (s2)
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
*                       ## 30.opt.: output formats: p,n,a,s,v,u,t,m,k,g,* (see docs) (s7)
                       ## 31.opt.: maj. base call at depth>x<mindepth (def.x=mindepth) (s5)
50                       ## 32.opt.: keep trimmed reads (def=0). Enter min length.    (s2)
                       ## 33.opt.: max stack size (int), def= max(500,mean+2*SD)    (s3)
2                       ## 34.opt.: minDerep: exclude dereps with <= N copies, def=1 (s3)
                       ## 35.opt.: use hierarchical clustering (def.=0, 1=yes)      (s6)
                       ## 36.opt.: repeat masking (def.=1='dust' method, 0=no)      (s3,s6)
                       ## 37.opt.:  max threads per job (def.=6; see docs)   (s3,s6)
```

Follow the tutorial… 
2. Quality filtering.  (23 April, 19:25-23:21  4hours)
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 2
```

3. Within-sample clustering (~2hrs/lane)   (23:21- 10:34, 24apr, 11hours)
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 3
```

Steps 4: Joint estimate of H and error (~9hrs/lane) (4:15 26Apr,    42hrs)
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 4
```

Steps 5: Consensus base calling (~1.5hrs/lane) (26Apr, 14:41;   10.5hrs)
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 5  
```

Step 6: Across-sample clustering  (22:30, 26Apr;   8hrs)
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 6
```

>>>Note that there was a bug with pyrad, so that s7 would not work. I had to wait for the bug fix before I could run it. 
Herbert updated pyrad on the fgcz server first. I have transferred the folder (c94d6H1.9) to fgcz to run s7. 

*5 May 2015*

The bug fix didn’t work. I’m not sure what the problem is now. 
I ran one of the earlier versions of pyrad since it was run on the gdcsrv1. I could try to run the latest  version of pyrad on the gdc server, as Stefan has just updated it. 

Waiting for Tim’s run to finish.. Should be in a few hours time. 


###Step 7: Statistics and output files
```
%%bash
/usr/local/ngseq/src/pyrad-3.0.4/pyRAD -p params.txt -s 7
```

Reruns of pyrad
I can’t find the problem with the pyrad run. But it has been updated to v3.0.6 on the GDC and FGCZ servers. 

I am rerunning on srv2 (because Christine was using srv1) and now srv 1 on the whole dataset of H1-9 (430indivs)

Srv1 job started @ 23:08 on 14May 2015


After a lot of struggling to get anything to work, or to find the problem, I have restarted pyRAD on srv1 (20clusters) on Sunday 17 May around 22:00
And srv2 (12 clusters) on Thursday 13 May - Sun 31 May. (2days/lane of data)

The problem was that #13 wasn’t set properly. This is a mandatory parameter - a filter for excess paralogues, by setting a hard filter for He above threshold x. 

I’ve set this ridiculously high, since I’m worried that I’ll be losing loci due to population structure. 

Next step: 
SNP filtering with VCFtools. 

##SNP filtering
I’m using VCFtools for filtering. v. 1.12b is on the gdcserver. 

- Documentation for VCFtools: http://vcftools.sourceforge.net/perl_module.html#vcf-filter
- For VCFtools filtering options: http://vcftools.sourceforge.net/man_0112b.html

For the initial run, I will use the ###ddocent tutorial: https://github.com/jpuritz/dDocent/blob/master/tutorials/Filtering%20Tutorial.md


To make this file more manageable. Let's start by a three step filter. We are going to only keep variants that have been successfully genotyped in 50% of individuals, a minimum quality score of 30, and a minor allele count of 3.

```
vcftools --vcf outfiles/H1-9_c94d6.vcf --max-missing 0.5 --mac 3 --recode --recode-INFO-all --out raw.g5mac3

Parameters as interpreted:
	--vcf outfiles/H1-9_c94d6.vcf
	--recode-INFO-all
	--mac 3
	--max-missing 0.5
	--out raw.g5mac3
	--recode

Eighth Header entry should be INFO: INFO    
After filtering, kept 432 out of 432 Individuals
Outputting VCF file...
After filtering, kept 55356 out of a possible 504183 Sites
Run Time = 149.00 seconds
```


```
vcftools --vcf raw.g5mac3.recode.vcf --minDP 3 --recode --recode-INFO-all --out raw.g5mac3dp3 

Parameters as interpreted:
	--vcf raw.g5mac3.recode.vcf
	--recode-INFO-all
	--minDP 3
	--out raw.g5mac3dp3
	--recode

After filtering, kept 432 out of 432 Individuals
Outputting VCF file...
After filtering, kept 55356 out of a possible 55356 Sites
Run Time = 41.00 seconds
```

Creates a .imiss file with missing data per individual. 
```
vcftools --vcf raw.g5mac3dp3.recode.vcf --missing-indv 
```

*10 June 2015*

to draw a histogram of this: 
	1. first make a file with only the missing data proportions included

```
mawk ‘!/IN/‘ out.imiss | cut -f5 > totalmissing
```
	2. plot using gnuplot
```
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

For H1-9 the data looks like this: 
##insert graph here




So this means that most of the missing data is below 50%. I’ll make 50% my cut-off for now. Some individuals have dropped out completely. 


1. First, make a list of all the individuals with >50% missing data
```
mawk ‘$5 > 0.5’ out.imiss | cut -f1 > lowDP.indiv
```

2. count the number of indivs in the file
```
wc -l lowDP.indiv
26
```

3Calculate the % 
	-scale tells you how many decimal places
```
(echo scale=3; echo "26/430") | bc
0.060
```

So if I filter for 50% missingness, I lose 26/430 indivs = 6%


Now we use *vcf tools* to filter out these individuals. 
```
vcftools --vcf raw.g5mac3.recode.vcf --remove lowDP.indiv --recode --recode-INFO-all --out raw.g5mac3dplm
```

1. make a file with population information
	-cut all the individual names into a file
```
mawk ‘$1’ out.imiss | cut -f1 > popmap
```
	- for the second column cut the population name out of the individual name
	and add this as a second column in popmap
```
cut -c-4 out.imiss > pop
	**useful info about cut here: http://www.thegeekstuff.com/2013/06/cut-command-examples/
paste popmap pop > popfile
```
	but now the headings for both columns is INDIV. To change this
```
awk '$1=="INDV"{$2="POP"}1' popfile > popfile2
```

#Now according to the tutorial, we should filter for loci with >10% missing data. I will try this and see how many loci are left. 
```
vcftools --vcf raw.g5mac3dplm.recode.vcf --max-missing 0.10 --maf 0.05 --recode --recode-INFO-all --out DP3g95maf05 --min-meanDP 20

Parameters as interpreted:
	--vcf raw.g5mac3dplm.recode.vcf
	--recode-INFO-all
	--maf 0.05
	--min-meanDP 20
	--max-missing 0.1
	--out DP3g95maf05
	--recode

After filtering, kept 407 out of 407 Individuals
Outputting VCF file...
After filtering, kept 0 out of a possible 55356 Sites
No data left for analysis!
Run Time = 5.00 seconds
```

Okay, so I have more than 10% missing data across all loci. So perhaps I can do a similar graph of the proportion of missing data as for the per individual missing data. 
But this is a bit more difficult, because it’s something I’ll have to calculate. And I’m not sure how missing data is coded. (N?)

->Changed the max missing data to 40% and then to 60%, but I’m still filtering out all of the loci. So something else is wrong here. 

->Actually I think it was the Min-DP 20 flag that was probably filtering everything out. 

->Yep

->So filtering for 5% missing data: (this is the same up to 40% missing, since I filtered for this during the pyrad run)

```
vcftools --vcf raw.g5mac3dplm.recode.vcf --max-missing 0.05 --maf 0.05 --recode --recode-INFO-all --out DP3g95maf05

VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf raw.g5mac3dplm.recode.vcf
	--recode-INFO-all
	--maf 0.05
	--max-missing 0.05
	--out DP3g95maf05
	--recode

After filtering, kept 407 out of 407 Individuals
Outputting VCF file...
After filtering, kept 26289 out of a possible 55356 Sites
Run Time = 15.00 seconds
```

I end up with #~26000 SNPs
(pretty good!) This means one locus every ~140Mb. Not amazing resolution, but still alright. 

->so most of the rest of the filters in the tutorial were already applied during the pyrad run (depth, quality.. ) and the rest are for paired sequences, so are irrelevant. 

->Final filter is for HWE. This needs to be per population (although this assumes a priori knowledge of population structure.. )

*PCA*
-I will fist do a PCA on the SNPs that I have and then run the filter for HWE



-For sampling with low depth and many individuals, it makes sense to incorporate probabilistic models into calling loci rather than hard filters. This can be applied to data from de Novo assembly or from mapping. Some packages from the Nielsen lab have recently been released: 
ngsTools and ANGSD

->I’ve done the basic filtering which they require: i.e. remove bad individuals and spurious loci - apart from the HWE filter. 
->So now I will use the ngsTools to run some basic pop stats. 

After deNovo assembly (or mapping) and basic filtering, use the program ANGSD to calculate genotype likelihoods. 

ANGSDwebsites: https://github.com/ANGSD/angsd
Tutorial: http://cgrlucb.wikispaces.com/file/view/CGRL_SNP_workshop.pdf
ngsTools: https://github.com/mfumagalli/ngsTools/blob/master/TUTORIAL.md

		###I have no idea how to make this work with de novo data or with .vcf files!!

So, I will just try to get pair-wise Fst using vcftools for now. I need a file with individual names for each population
->First, split the population file (with indivs + pops) by population into different files
```
awk '{print > $2}' popfile 
```
no need to remove population names. 
run the Fst e.g. for 2 pops:
```
vcftools --vcf DP3g95maf05.recode.vcf --weir-fst-pop stba --weir-fst-pop bela --out stba_vs_bela
```

So, I can get this to work for 2 populations, but it just calculates an average fst when I input more pops. 
Still have no idea how to get from the filtered SNPs to basic summary stats… 











##Sensitivity analyses: 
1. SNPs vs nr of individuals (sensitivity analysis)
2. Nucleotide diversity vs nr individuals
3. Fst vs nr of individuals 


Basic comparisons: 
- Average and range of depth
- Depth across lanes
- Depth across populations
- SNPs per population
- Missing data per population


###Statistics
- PCA
- Hierarchical PCA
- Fst between these populations
- nucleotide diversity per population
- Nucleotide diversity per elevation



##3. Mapping tests

*3 April 2013*

Received the R.temporaria draft genome from Alan. This is the information he sent me: 

*The genome itself is pretty rough; it amounts to 35% of the expected genome size, and the scaffold n50 is only 924 bp. On the other hand, 98% of the CEGMA core genes are at least partially represented, so I think it includes most of the non-repetitive part of the genome, and the missing 3 Gbp is enriched for repeats. About 10% of the genome assembly can be assigned to a position in X. tropicalis.

Assembling the genome took quite a bit of time and effort, so if you decide to use it, I would like to be included as a coauthor on one paper. Is this something you and Josh are open to?*

File name: 
####Rtk43.fa.gz (473Mb)


I will run a test alignment with H2 on my mac laptop: 

Using information from http://ged.msu.edu/angus/tutorials-2013/bwa-tutorial.html

###bwa
```
brew install bwa
mkdir ~/NGSpipelines/mapping/bwaH2
cd ~/NGSpipelines/mapping/bwaH2
cp ~/phd_20150212/2_Data/Rtk43.fa.gz .
```

index the genome (this took 2342.221 sec - 39min)
```
bwa index Rtk43.fa.gz
```

Align (this took 245012.827 sec - 68hrs for 215Mil reads)
```
bwa aln Rtk43.fa.gz ~/phd_20150212/2_Data/20141219.B-p276_h2_R1.fastq > Rt.H2.sai
```

Convert the .sai file to SAM format
```
bwa samse Rtk43.fa.gz Rt.H2.sai ~/phd_20150212/2_Data/20141219.B-p276_h2_R1.fastq > Rt.H2.sai.sam
```


####Processing the BWA and Bowtie output for use with Samtools

Even the SAM file isn’t very useful unless we can get it into a program that generates more readable output or lets us visualize things in a more intuitive way. For now, we’ll get the output into a sorted BAM file so we can look at it using Samtools later.

*Problems with brew. permission errors when symlinking to files.* 

####Solution: http://developpeers.com/blogs/fix-for-homebrew-permission-denied-issues

```
~ phabi$ sudo chown -R `whoami` /usr/local
Password:
~ phabi$ brew link --overwrite node
Linking /usr/local/Cellar/node/0.10.4... 5 symlinks created
```

Install samtools: 
```
brew install samtools
```

Like bwa, Samtools also requires us to go through several steps before we have our data in usable form. First, we need to have Samtools generate its own index of the reference genome:
```
gunzip Rtk43.fa.gz
mkdir samtools
cd samtools
```

Index the genome using samtools -this took 2985.120 sec i.e. 50min
```
samtools faidx  ~/NGSpipelines/mapping/bwaH2/Rtk43.fa
```

Next, we need to convert the SAM file into a BAM file. (A BAM file is just a binary version of a SAM file.)  - ~40min
```
samtools import Rtk43.fa.fai Rt.H2.sai.sam Rt.H2.bam
```

Now, we need to sort the BAM file (time: 1h15)
```
samtools sort Rt.H2.bam Rt.H2.bam.sorted
```

And last, we need Samtools to index the BAM file (3min)
```
samtools index Rt.H2.bam.sorted.bam
```

###Optimise mapping parameters: 

Map individuals to the genome 26 - 28 May 2015
 
Tutorial from: http://2013-caltech-workshop.readthedocs.org/en/latest/bwa_mapping.html
*Aim:* mapping success per individual

1. map using bwa
2. call SNPs using GATK/FreeBayes

*I will map wdji11 and stls03 to Rt genome*

```
mkdir /gdc_home4/alexjvr/mapping/wdji11_map_20150524
```

Index the reference (00.25-01:23)
```
bwa index -a bwtsw Rtk43.fa
```

transfer wdji11.fq to the folder (demultiplexed file)
start bwa alignment using 2 cores (-t 2)  (9:17-9:47)
```
bwa aln -t 2 Rtk43.fa wdji_11.fq > wdji11_RtAln.sai
```

generate sam file (10:30-10:32)
```
bwa samse Rtk43.fa wdji1_RtAln.sai wdji_11.fq > wdji11_RtAln.sam
```

now use samtools to process the output into useful information
samtools needs to make it’s own index of the genome
```
samtools faidx Rtk43.fa
```
this outputs a samtools indexed genome: Rtk43.fa.fai

convert the .sam output from bwa to a bam file
```
samtools import Rtk43.fa.fai wdji11_RtAln.sam wdji11_RtAln.bam
```

sort the .bam file (this step takes a while.. be patient)
```
samtools sort wdji11_RtAln.bam wdji11_RtAln.sorted.bam
```

and index the .bam file (takes 2sec - output is xx.bai)
```
samtools index wdji11_RtAln.sorted.bam
```

##Variant calling
now we have the aligned and indexed genome, we have to call SNPs. Christine uses GATK. Others use FreeBayes… 

According to this FreeBayes is currently the better variant caller:
http://bcb.io/2013/10/21/updated-comparison-of-variant-detection-methods-ensemble-freebayes-and-minimal-bam-preparation-pipelines/

And here is a FreeBayes tutorial: 
http://clavius.bc.edu/~erik/CSHL-advanced-sequencing/freebayes-tutorial.html

I will try FreeBayes with default settings first
```
freebayes -f genome.fa \ alignment.sorted.bam >output.FB.vcf
freebayes -f Rtk43.fa \ wdji11_RtAln.sorted.bam >wdji11.Rtk.FB.vcf 
```











###2. map called SNPs to genome 
***https://docs.google.com/presentation/d/1ShbbQ-Y0umYRcHEgqiWDTy4l-E5qH0oZtz97ln52J5k/edit#slide=id.p9
Variant calling presentation with useful figures

*Aim*: to see how much of the variation in my SNPs if mappable (i.e. useful) 

I haven’t optimised mapping parameters yet, but just to get an indication of mapping success, I will use the same protocol as before. 
I will use the output from c94d6 and from the within population results (


*7 April 2015*

I now have access to the S3IT cluster: 
ssh alexjvr@130.60.24.131 
1Tb of memory available here

I will test the Alignment pipeline here using bowtie. 

Demultiplex H1 using process_radtags (no quality filtering)
```
/usr/local/ngseq/stow/stacks-1.28/bin/process_radtags -i gzfastq -f /srv/gstore4users/p1795/HiSeq_20150225_RUN171/20150225.A-H1_R1.fastq.gz -o ./H1 -y fastq -b ~/barcodes_RE --disable_rad_check -r -D
```

68% of reads kept. 
With ~20% PhiX. So 10% of reads lost due to multiple mutations..


###2. Rename files using the ddocent script:
```
rename.for.dDocent

if [ -z "$1" ]
then
echo "No file with barcodes and sample names specified."
echo "Correct usage: Rename_for_dDocent.sh barcodefile"
exit 1
else
NAMES=( `cut -f1  $1 `)
BARCODES=( `cut -f2 $1 `)
LEN=( `wc -l $1 `)
LEN=$(($LEN - 1))

echo ${NAMES[0]}
echo ${BARCODES[0]}

for ((i = 0; i <= $LEN; i++));
do
mv sample_${BARCODES[$i]}.1.fq ${NAMES[$i]}.F.fq
mv sample_${BARCODES[$i]}.2.fq ${NAMES[$i]}.R.fq
done
fi
```

Modify this code for SE:
```
if [ -z "$1" ]
then
echo "No file with barcodes and sample names specified."
echo "Correct usage: Rename_for_dDocent.sh barcodefile"
exit 1
else
NAMES=( `cut -f1  $1 `)
BARCODES=( `cut -f2 $1 `)
LEN=( `wc -l $1 `)
LEN=$(($LEN - 1))

echo ${NAMES[0]}
echo ${BARCODES[0]}

for ((i = 0; i <= $LEN; i++));
do
mv sample_${BARCODES[$i]}.1.fq ${NAMES[$i]}.F.fq
mv sample_${BARCODES[$i]}.2.fq ${NAMES[$i]}.R.fq
done
fi
```

Make a file for renaming the barcodes with the following format: 
```
pop_sample  barcode 

eg:
oalp_01	GCATGAATT
oalp_02	AACCAAATT
oalp_03	CGATCAATT
oalp_04	TCGATAATT
oalp_05	TGCATAATT
oalp_06	CAACCAATT
oalp_07	GGTTGAATT
oalp_08	AAGGAAATT
oalp_09	AGCTAAATT
oalp_10	ACACAAATT
```
Run the script:
```
perl rename.for.dDocent barcodefile
```
eg:
```
perl rename.for.dDocent H1.barcodesRE.samplenames
```

##3. Mapping

After reading Hatem et al. 2013, I decided to try Bowtie2 for mapping. This is similar in mapping to BWA, but should run slightly faster. 

Using tutorial from here: http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml#performance-tuning

Index the genome
```
mkdir H1_bowtie2
cd H1_bowtie2
gunzip Rtk43.fa.gz
/usr/local/ngseq/bin/bowtie2-build ~/Rtk43.fa Rtemp43_indexed
```

start 22:56


Read alignment (and for loop to align all sequences in the demultiplexed folder. Run this from the home directory with:
a folder containing all the renamed demultiplexed files (e.g. H1/oalp_01.fq…)
a folder containing the indexed genome with 4 files from the bowtie2 indexing output (here H1_bowtie2/Rtemp43_indexed)
fastq is the default file format, so -q not neccessary
```
for x in H1/*.fq; do /usr/local/ngseq/bin/bowtie2 -x H1_bowtie2/Rtemp43_indexed -U ${x} -S ${x}.sam; done
```

Started *10 April 15:20*
Runs ~25min per sample on the S3IT server. So, for 48 samples ~20hours

Moved everything to the gdc server after S3IT ran out of space. 


Once all .sam files have been generated, process with samtools: 

Index genome with samtools
```
mkdir samtoolsH1
mv Rtk43.fa samtoolsH1/
cd samtoolsH1/
samtools faidx Rtk43.fa
```

Convert all .sam to .bam
```
for x in ~/mapping/H1/done/*sam; do samtools import Rtk43.fa.fai ${x} ${x}.bam ; done
```

Sort files (move to directory containing .sam.bam files)
```
for x in *.sam.bam
do samtools sort ${x} ${x}.sorted
done
```

Index the reads
```
for x in *.sorted.bam; do samtools index ${x}; done
```



##Pyrad optimisation

Optimal clustering threshold (Ilut et al. 2014) e.g. Barley et al. 2015
Sensitivity analysis for 
1. min percentage indivs in a population with the locus
2. min number of pops to have a locus
3. minimum stack depth





###Prep for meeting with Josh 25 May 2015

1. pyrad optimization

2. map of samples chosen for optimisation

3. pop genetics
I will use the output from the c94d6 run found here: /gdc_home4/alexjvr/pyradOpt/c94d6_done
This has the 28 indivs from 14 populations from the pyrad optimisation run 

I will try to do a PCA using Eigensoft. 
Info from Tim: 
*[25/05/15 11:56:50] braytc: use PGDSpider to convert from vcf to eigen
all three eigen input files are necessary (.ind .geno .snp)
then use eigenfiles directly
(be careful of reorder of individuals)
outputs are .evec .eval
inside the POP... folder you need to have the .perl and the .example altered to reflect the file names where required
you will need to work from the examples given
as long as eigensoft is added to path you can then just call your modified perlscript;
./filename.perl
[25/05/15 11:57:59] braytc: if you have trouble then send me the vcf and i can try to do it*


convert vcf output file to eigen using PGDSpider


###Population Genetics

####Structure

Command line Structure needs the mainparams and extraparams files to run. Here all the run parameters are specified. 

Several K’s can be run simultaneously: 
```
structure -K 1 -o output_1.1.txt; structure -K 1 -o output_1.2.txt; structure -K 1 -o output_1.3.txt; structure -K 1 -o output_1.4.txt; structure -K 1 -o output_1.5.txt; structure -K 2 -o output_2.1.txt; structure -K 2 -o output_2.2.txt; structure -K 2 -o output_2.3.txt; structure -K 2 -o output_2.4.txt; structure -K 2 -o output_2.5.txt; structure -K 3 -o output_3.1.txt; structure -K 3 -o output_3.2.txt; structure -K 3 -o output_3.3.txt; structure -K 3 -o output_3.4.txt; structure -K 3 -o output_3.5.txt; structure -K 4 -o output_4.1.txt; structure -K 4 -o output_4.2.txt; structure -K 4 -o output_4.3.txt; structure -K 4 -o output_4.4.txt; structure -K 4 -o output_4.5.txt
```


Download the output and open in Structure gui

Templates: 
mainparams: 
```
#KEY PARAMETERS FOR THE PROGRAM structure.  YOU WILL NEED TO SET THESE
#IN ORDER TO RUN THE PROGRAM.  VARIOUS OPTIONS CAN BE ADJUSTED IN THE
#FILE extraparams.


#"(int)" means that this takes an integer value.
#"(B)"   means that this variable is Boolean 
        (ie insert 1 for True, and 0 for False)
#"(str)" means that this is a string (but not enclosed in quotes!) 

#Data File

#define INFILE  c98d6m4p3.str   // (str) name of input data file
#define OUTFILE results  //(str) name of output data file

#define NUMINDS   28    // (int) number of diploid individuals in data file
#define NUMLOCI   5390   // (int) number of loci in data file
#define LABEL     1     // (B) Input file contains individual labels
#define POPDATA   0     // (B) Input file contains a population 
                             identifier
#define POPFLAG   0     // (B) Input file contains a flag which says 
                              whether to use popinfo when USEPOPINFO==1
#define PHENOTYPE 0     // (B) Input file contains phenotype information
#define EXTRACOLS 0     // (int) Number of additional columns of data 
                             before the genotype data start.
#define PHASEINFO 0     // (B) the data for each individual contains a line
                                  indicating phase
#define MARKOVPHASE  0  // (B) the phase info follows a Markov model.

#define MISSING      -9 // (int) value given to missing genotype data
#define PLOIDY       2  // (int) ploidy of data

#define ONEROWPERIND 0  // (B) store data for individuals in a single line
#define MARKERNAMES  0  // (B) data file contains row of marker names
#define MAPDISTANCES 0  // (B) data file contains row of map distances 
                             // between loci


Program Parameters

#define MAXPOPS    2    // (int) number of populations assumed
#define BURNIN    10000 // (int) length of burnin period
#define NUMREPS   10000 // (int) number of MCMC reps after burnin

Command line options:

-m mainparams
-e extraparams
-s stratparams
-K MAXPOPS 
-L NUMLOCI
-N NUMINDS
-i input file
-o output file

			

extraparams:
EXTRA PARAMS FOR THE PROGRAM structure.  THE MOST IMPORTANT PARAMS ARE
IN THE FILE mainparams.

"(int)" means that this takes an integer value.
"(d)"   means that this is a double (ie, a Real number such as 3.14).
"(B)"   means that this variable is Boolean 
        (ie insert 1 for True, and 0 for False).

PROGRAM OPTIONS

#define FREQSCORR   1 // (B) allele frequencies are correlated among pops
#define ONEFST      0 // (B) assume same value of Fst for all subpopulations.

#define INFERALPHA  1 // (B) Infer ALPHA (the admixture parameter)
#define POPALPHAS   0 // (B) Individual alpha for each population

#define INFERLAMBDA 0 // (B) Infer LAMBDA (the allele frequencies parameter)
#define POPSPECIFICLAMBDA 0 //(B) infer a separate lambda for each pop 
					(only if INFERLAMBDA=1).

#define NOADMIX     0 (B) Use no admixture model
#define LINKAGE     0 // (B) Use the linkage model model 
#define PHASED      0 // (B) Data are in correct phase (required unless data are diploid)
                      If (LINKAGE=1, PHASED=0), then PHASEINFO can be used--this is an
		      extra line in the input file that gives phase probabilities.
                      When PHASEINFO =0 each value is set to 0.5, implying no phase information.

#define LOG10RMIN     -4.0   //(d) Log10 of minimum allowed value of r under linkage model
#define LOG10RMAX      1.0   //(d) Log10 of maximum allowed value of r
#define LOG10RPROPSD   0.1   //(d) standard deviation of log r in update
#define LOG10RSTART   -2.0   //(d) initial value of log10 r

#define COMPUTEPROB 1 // (B) Estimate the probability of the Data under 
                             the model.  This is used when choosing the 
                             best number of subpopulations.

#define ADMBURNIN   500 //(int) initial period of burnin with admixture model. 
			The admixture model normally has the best mixing properties,
			and therefore should be used as the first phase of the burnin.
                        A significant burnin is highly recommended under the linkage
                        model which otherwise can perform very badly.
                         

USING PRIOR POPULATION INFO

#define USEPOPINFO  0  // (B) Use prior population information to assist 
                             clustering.  Need POPDATA=1.
#define GENSBACK    1  //(int) For use when inferring whether an indiv-
                         idual is an immigrant, or has an immigrant an-
                         cestor in the past GENSBACK generations.  eg, if 
                         GENSBACK==2, it tests for immigrant ancestry 
                         back to grandparents. 
#define MIGRPRIOR 0.001 //(d) prior prob that an individual is a migrant 
                             (used only when USEPOPINFO==1).  This should 
                             be small, eg 0.01 or 0.1.
#define PFROMPOPFLAGONLY 0 // (B) only use individuals with POPFLAG=1 to update	P.
                                  This is to enable use of a reference set of 
                                  individuals for clustering additional "test" 
                                  individuals.

OUTPUT OPTIONS

#define PRINTKLD     1 // (B) Print estimated Kullback-Leibler diver-
                              gence to screen during the run
#define PRINTLAMBDA  1 // (B) Print current value(s) of lambda to screen
#define PRINTQSUM    1 // (B) Print summary of current population membership to screen



#define SITEBYSITE   0  // (B) whether or not to print site by site results. Large!

#define PRINTQHAT    0  // (B) Q-hat printed to a separate file.  Turn this 
                           on before using STRAT.
#define UPDATEFREQ   0  // (int) frequency of printing update on the screen.
                                 Set automatically if this is 0.
#define PRINTLIKES   0  // (B) print current likelihood to screen every rep
#define INTERMEDSAVE 0  // (int) number of saves to file during run

#define ECHODATA     1  // (B) Print some of data file to screen to check
                              that the data entry is correct.
(NEXT 3 ARE FOR COLLECTING DISTRIBUTION OF Q:)
#define ANCESTDIST   0  // (B) collect data about the distribution of an-
                              cestry coefficients (Q) for each individual
#define NUMBOXES   1000 // (int) the distribution of Q values is stored as 
                              a histogram with this number of boxes. 
#define ANCESTPINT 0.90 //(d) the size of the displayed probability  
                              interval on Q (values between 0.0--1.0)

PRIORS
#define ALPHA      1.0  // (d) Dirichlet parameter for degree of admixture 
                             (this is the initial value if INFERALPHA==1).
#define FPRIORMEAN 0.01 // (d) Prior mean and SD of Fst for pops.
#define FPRIORSD   0.05  // (d) The prior is a Gamma distribution with these parameters

#define LAMBDA      1.0 // (d) Dirichlet parameter for allele frequencies 
#define UNIFPRIORALPHA 1 // (B) use a uniform prior for alpha;
                                otherwise gamma prior
#define ALPHAMAX     20.0 // (d) max value of alpha if uniform prior
#define ALPHAPRIORA  0.05 // (only if UNIFPRIORALPHA==0): alpha has a gamma 
                            prior with mean A*B, and 
#define ALPHAPRIORB  0.001 // variance A*B^2.  Suggest A=0.1, B=1.0

MISCELLANEOUS

#define ALPHAPROPSD 0.025 //(d) SD of proposal for updating alpha
#define STARTATPOPINFO 0 //Use given populations as the initial condition 
                           for population origins.  (Need POPDATA==1).  It 
                           is assumed that the PopData in the input file 
                           are between 1 and k where k<=MAXPOPS.
#define RANDOMIZE    1  //(B) use new random seed for each run 
#define METROFREQ    10  //(int) Frequency of using Metropolis step to update
                           Q under admixture model (ie use the metr. move every
                           i steps).  If this is set to 0, it is never used.
                           (Proposal for each q^(i) sampled from prior.  The 
                           goal is to improve mixing for small alpha.)
#define REPORTHITRATE 0 //(B) report hit rate if using METROFREQ
```

###10 July 2015

I just recieved the next flowcell of data: 

H10, H13, H14, H16, H17, H18, H19, H21

Fastqc has been fun for me by the FGCZ bioinformaticians: http://fgcz-gstore.uzh.ch/projects/p1795/QC_Fastqc__2015-07-07--14-02-11/FastQC_Result/

Library | Number of Sequences
:--:|:--:
H10 | 103020694
H13 | 218264182
H14 | 217765320
H16 | 237732001
H17 | 220987734
H18 | 222052040
H19 | 189469463
H21 | 104792211


###7 Oct

I've been running de Novo assembly on 530 CH individuals. It is possible to split pyRAD steps 2-5 between servers. I've been using GDC servers 1 & 2. This reduces the time by ~50%

I've reduced the allowed amount of missing data to 100/530 (20%). I will analyse the data in subsets. 

My task this week is to use the new CH530 dataset to work out how to do the following: 

1. SNP filtering - what to filter & how to optimise this
2. Summary stats (Heterozygosity, nucleotide diverstiy, etc)
3. How many individuals per population until max information is reached?
3. PCA
4. Structure
5. TESS
6. AMOVA
7. IBD
8. F statistics
9. PCAdapt
10. Fst outlier
11. EMMAX
12. Multivariate analysis
 


####1. SNP filtering

We're using VCFtools and filtering for the following things:

1. Sequence quality
2. MAF
3. Missingness within individuals
4. Missingness across loci (remember to do this with different subsets of the data)
5. HWE within populations *** Think about this step before applying it to your data
6. 

In the end we want several datasets: 

1. CHN
2. CHS
3. CHtotal
4. CH_4gradients
5. SE

First filter for MAF of 3 and sequence quality of 30 (I'm not filtering for genotyping rate at this point)

```
vcftools --vcf CHc94d6min530.vcf --mac 3 --recode --recode-INFO-all --out raw.530mac3


VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf CHc94d6min530.vcf
	--recode-INFO-all
	--mac 3
	--out raw.530mac3
	--recode

Eighth Header entry should be INFO: INFO    
After filtering, kept 530 out of 530 Individuals
Outputting VCF file...
After filtering, kept 205921 out of a possible 334483 Sites
Run Time = 90.00 seconds
```

Min Depth of 3 for the genotype **I already filter for minDP of 6 in the pyRAD run. I should decrease this to 3 to increase the number of loci! I need to rerun this pyRAD!

```
vcftools --vcf raw.530mac3.recode.vcf --minDP 3 --recode --recode-INFO-all --out raw.530mac3dp3

VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf raw.530mac3.recode.vcf
	--recode-INFO-all
	--minDP 3
	--out raw.530mac3dp3
	--recode

After filtering, kept 530 out of 530 Individuals
Outputting VCF file...
After filtering, kept 205921 out of a possible 205921 Sites
Run Time = 74.00 seconds

```

And now filter for individuals that sequenced badly

Assess the amount of missing data (note the change in the --missing command between v0.1.11 & 0.1.12b)
```
vcftools --vcf raw.530mac3dp3.recode.vcf --missing-indv

cat out.imiss
```

Draw a histogram of these results

```
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

                                         Histogram of % missing data per individual
  Number of Occurrences
    30 ++--------------+--------------+---------------+---------------+---------------+--------------+--------------++
       +               +              +               +        'totalmissing' using (bin( $1,binwidth)):(1.0) ****** +
       |                                                                                                             |
       |                                     **                                                                      |
    25 ++                                    **                                                                     ++
       |                                     **                                                                      |
       |                   ***     ****   ** **                                                                      |
       |                   * * ***** **   ** **                                                                      |
    20 ++                  * * ** ** **   ** **                                                                     ++
       |                   * * ** ** **   **********                                                                 |
       |                  ** **** ** ** **** ** ** ****                                                              |
       |                **** * ** ** **** ** ** ** * **                                                              |
    15 ++               * ** * ** ** ** * ** ** ** * **                                                             ++
       |                * ** * ** ** ** * ** ** ** * **                                                              |
       |            ***** ** * ** ** ** * ** ** ** * ****                                           **               |
       |            ** ** ** * ** ** ** * ** ** ** * ** *                                           **               |
    10 ++           ** ** ** * ** ** ** * ** ** ** * ** *                                           **              ++
       |            ** ** ** * ** ** ** * ** ** ** * ** **                                          **               |
       |            ** ** ** * ** ** ** * ** ** ** * ** ****                                        **               |
       |        *** ** ** ** * ** ** ** * ** ** ** * ** ** ****                                     **               |
     5 ++ ******* * ** ** ** * ** ** ** * ** ** ** * ** ** ** *       **                            **              ++
       |  * ** ** **** ** ** * ** ** ** * ** ** ** * ** ** ** ****    **                          ****               |
       | ** ** ** * ** ** ** * ** ** ** * ** ** ** * ** ** ** * ** ** ***** ******* ***   *****   * **               |
       **** ** ** * ** ** ** * ** ** ** * ** ** ** * ** ** ** * ********  ***  *  *** ***** * ***** ****             +
     0 *************************************************************************************************------------++
      0.4             0.5            0.6             0.7             0.8             0.9             1              1.1
                                                      % of missing data

Most of the missing data is below 70%. So I will make the cutoff 70% for now. 

First create a list of all the individuals that will be removed. And look at the file
```
mawk '$5 > 0.7' out.imiss | cut -f1 > lowDP.indv

cat lowDP.indv

wc -l lowDP.indv
```

This will remove 95 individuals (of 530 ~18%)

pozz, seeo, & orge are the only 3 populations that drop below 5 indivs/pop after filtering, so I will go ahead. 

```
vcftools --vcf raw.530mac3dp3.recode.vcf --remove lowDP.indv --recode --recode-INFO-all --out subset.imiss70

VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf raw.530mac3dp3.recode.vcf
	--exclude lowDP.indv
	--recode-INFO-all
	--out subset.imiss70
	--recode

Excluding individuals in 'exclude' list
After filtering, kept 436 out of 530 Individuals
Outputting VCF file...
After filtering, kept 205921 out of a possible 205921 Sites
Run Time = 57.00 seconds
```

And now I can filter for missing data and maf per locus. Previously I did this for 10% missing data. Let's see what happens with the overall dataset
```
vcftools --vcf subset.imiss70.recode.vcf --max-missing 0.10 --maf 0.05 --recode --recode-INFO-all --out CHall436locmiss10

VCFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf subset.imiss70.recode.vcf
	--recode-INFO-all
	--maf 0.05
	--max-missing 0.1
	--out CHall436locmiss10
	--recode

After filtering, kept 436 out of 436 Individuals
Outputting VCF file...
After filtering, kept 100633 out of a possible 205921 Sites
Run Time = 40.00 seconds
```

So, I lose 50% of the data, but I still have way more loci than I did with the subset data! (137 indivs, After filtering, kept 16478 out of a possible 25932 Sites). Can this be right? I suppose there are multiple SNPs per locus. How many loci are there? 
at 94% clustering, assuming ~120bp final reads, the max number of SNPs per cluster is 7. This is pretty high. The default in pyRAD is 100SNPs allowed. I will have to check on this. 


Finally, filter for within population HWE using a script recommended by Jon Puritz (in his SNPfiltering tutorial on GitHub)

```
curl -L -O https://github.com/jpuritz/dDocent/raw/master/scripts/filter_hwe_by_pop.pl
chmod +x filter_hwe_by_pop.pl

filter_hwe_by_pop.pl -v CHall436locmiss10.recode.vcf -p popmap530 -o CHall436locmiss10.HWE -h 0.001
```
No loci filtered for any of the populations. ** Christine doesn't filter for HWE, as this may remove interesting loci. 




####2. Summary Statistics

Nucleotide diversity

We can calculate per site nucleotide divergence in vcftools
```
vcftools --vcf CHall436locmiss10.recode.vcf --site-pi --out CH436_pi
```


Heterozygosity
Homozygosity
HWE
Fis



And then I need to split the dataset up into CHN, CHS, and CH4grad. I can generate these by defining all the excluded individuals for s7 (line 17 in params.txt) for pyRAD. 



##12 Oct

vcftools --het option calculates the Method of Moments inbreeding statistic: 

https://www.cog-genomics.org/plink2/basic_stats

```
vcftools --vcf CHall436locmiss10.recode.vcf --het
```

Save the output as .csv and add population information in (elevation, class, canton). 

Plot these results as F/pop in R. 

Import the data set, and then: 

```
#Plot F (MOM)

CH <- CH436.het.out


require(ggplot2)
qplot(CH$Canton, CH$F, fill=factor((Elev.Class), levels=c("Low","Mid","High")), data=CH, geom="boxplot", position="dodge") + xlab("Transect") + ylab("Method of Moments F statistic")+ ggtitle("Method of Moments F for each elevational transect in Switzerland") + scale_fill_manual(values=c("lightgoldenrod1","olivedrab2","royalblue"))+theme_bw()+theme(legend.title=element_blank())


```


Now I will try to read my data into R

First convert it into plink and then recode: 
```
vcftools --vcf CHall436locmiss10.recode.vcf --out CH436.Final.plink --plink


CFtools - v0.1.12b
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf CHall436locmiss10.recode.vcf
	--out CH436.Final.plink
	--plink

After filtering, kept 436 out of 436 Individuals
Writing PLINK PED and MAP files ... 
	PLINK: Only outputting biallelic loci.
Done.
After filtering, kept 100633 out of a possible 100633 Sites
Run Time = 17.00 seconds


plink --file CH436.Final.plink --recodeA


Options in effect:
	--file CH436.Final.plink
	--recodeA

99355 (of 99355) markers to be included from [ CH436.Final.plink.map ]
Warning, found 436 individuals with ambiguous sex codes
Writing list of these individuals to [ plink.nosex ]
436 individuals read from [ CH436.Final.plink.ped ] 
0 individuals with nonmissing phenotypes
Assuming a disease phenotype (1=unaff, 2=aff, 0=miss)
Missing phenotype value is also -9
0 cases, 0 controls and 436 missing
0 males, 0 females, and 436 of unspecified sex
Before frequency and genotyping pruning, there are 99355 SNPs
436 founders and 0 non-founders found
Total genotyping rate in remaining individuals is -nan
0 SNPs failed missingness test ( GENO > 1 )
0 SNPs failed frequency test ( MAF < 0 )
After frequency and genotyping pruning, there are 99355 SNPs
After filtering, 0 cases, 0 controls and 436 missing
After filtering, 0 males, 0 females, and 436 of unspecified sex
Writing recoded file to [ plink.raw ] 

```


Now copy this over to the mac, and read into R as a genlight object. 
```
##Bash
scp -r alexjvr@gdcsrv1.ethz.ch:/gdc_home4/alexjvr/CH530SNPfiltering/filteredforR/CH436.plink.raw .

##R
setwd("~/phd_20150212/Analysis/ddRAD/CH530ddRAD/basicstats")

read.PLINK("CH436.plink.raw")
```

now we can use AdeGenet and Ape, etc. 




##8 OCT 2015

Trying to run PCAdapt on my filtered SNPs (CH530)

http://membres-timc.imag.fr/Michael.Blum/PCAdapt/pcadapt_markdown.html

- I installed the package in Rstudio

- I had to upgrade R for the package to run. See here for upgrading R: http://stackoverflow.com/questions/13656699/update-r-using-rstudio

- Also installed the package in terminal since they suggest using pcadapt in terminal for >100 000 loci. 

- my filtered dataset contains: 100633 sites (less loci, since multiple SNPs per locus)

PCAdapt

First convert my vcf file to pcadapt format

```
#in the folder with the input files:
/Users/alexjvr/phd_20150212/Analysis/ddRAD/CH530ddRAD

/Users/alexjvr/Software/pcadapt/PCAdaptPackage/vcf2pcadapt CHall436locmiss10.recode.vcf CH530filetered.pcadapt

#Run PCAdapt fast

/Users/alexjvr/Software/pcadapt/PCAdaptPackage/PCAdapt fast -i CH530filetered.pcadapt -K 4 -o CH530_K4
```

The optimal K needs to be chosen (number of latent factors). Do this by running several K's, and investigating the mean squared error as a function of K. The desired figures can be obtained by running the R script get_errors.R

I ran the script for K 1-10

And then adjust the Rscript from the pcadapt package to read the highest K run in R

```
R
source("/Users/alexjvr/Software/pcadapt/PCAdaptPackage/Rscripts/Display_Sigma_K.R")
```

This brings up a scree plot for CH530_K10


![alt_txt][screeplotK10]
[screeplotK10]:https://cloud.githubusercontent.com/assets/12142475/10367838/ed6430a6-6dd2-11e5-8862-9d70d164e1d4.png

It's not so clear to me what the K is from this plot. The authors suggest that (for the pcadapt fast run) K should be chosen just before K plateaus. So is this K=3? or K=7??

I'm not sure what to make of these results just yet. Most of the variation seems to be in PC1. But I can't draw the graph (only for up to PC4). I will come back to this. 

##Note 9 Oct 2015

For some reason I have only 530 individuals in the last pyRAD run i.s.o the 826 that have already been sequence. Once the last lanes of sequencing are out, I will rerun the entire pyRAD pipeline from scratch (including demultiplexing, concatenating H10, H21-23 (all sequenced twice. We're paying only for H21). Check all the barcode files to make sure that they are updated to the latest sample orders!!


So, for now I will continue to work on my current dataset. 

Next task: basic stats using Adegenet

https://github.com/thibautjombart/adegenet/wiki/Tutorials


1. First I need to convert vcf to fstat (or structure or genpop..). I'm using pgdspider for this. I also want to include a list of the populations. 

To get a list of all the names in the vcf file, use the following code

```
#!/bin/bash
set -eu
set -o pipefail
VCF="variants.vcf"
INDIVIDUALS=$( fgrep -m 1 '#CHROM' "$VCF" | cut -f10- | tr '\t' '\n' )
for IND in $INDIVIDUALS; do
echo $IND
done
```

Run PGDSpider through the command line (The GUI failed after ~30min due to insufficient memory) 
start 17:56. Both the GUI and the command line keep running out of memory. So I will need to subsample markers. 

```
java -Xmx1024m -Xms512m -jar PGDSpider2-cli.jar -inputfile /Users/alexjvr/phd_20150212/Analysis/ddRAD/CH530ddRAD/CHall436locmiss10.recode.vcf -inputformat VCF -outputfile /Users/alexjvr/phd_20150212/Analysis/ddRAD/CH530ddRAD/CH436.structure -outformat STRUCTURE -spid /Users/alexjvr/phd_20150212/Analysis/ddRAD/CH530ddRAD/vcftofstat.spid 
```





For the summary stats, Christine recommends the following: 

- Try different parameters and see if it makes a difference. She looked mostly at missingness. 

- For PCA filter for missingness across loci and populations. Missingness can affect the PCA

- for Fstats also filter for missingness. Missingness should only make the measures a bit more noisy. 

In R, when trying different plots, the parameter variables can be written to the plot using the following command: 

> sys.getenv ##Imports the variable in R and can use it to plot


Use VCFtools to filter the data further: 





