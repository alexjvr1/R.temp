###20Aug2015

##Task 1: SE RAD data

- I need to check how well the SE samples worked. 

Currently the H22 & H23 data that I have were from a failed run. (Cathy said there was something wrong with the PhiX). But since I cannot wait much longer, I will use the available samples to see if my PyRAD pipeline works. 

1. Check whether pyRAD protocol works
2. Test mapping percentage to *R.temp* draft genome
3. Compare markers with CH markers. 
4. Basic statistics: genetic diversity and divergence between pops

Available data are from two Skaone populations: SL & Ho



Sample|RawReads|AdaptDimer|%|FinalReads
:--:|:--:|:--:|:--:|:--:
H0_01|1457313|11401|0.78|1445912
H0_03|1258050|17692|1.41|1240358
H0_04|1067225|15315|1.44|1051910
H0_05|1179749|16364|1.39|1163385
H0_06|924094|11973|1.30|912121
H0_07|990095|13555|1.37|976540
H0_08|1272407|17558|1.38| 1254849
H0_09| 274656|3432|1.25|271224
H0_10|1330213|18915|1.42|1311298
H0_11|1047483|14519|1.39|1032964
H0_12|2477574|20775|0.84|2456799
H0_13|1789857| 13905|0.78|1775952
H0_14|1739633|14375|0.83|1725258
H0_15|3096212|24424 |0.79|3071788
H0_16|2659502 |22246|0.84|2637256
H0_17| 2396102|21842 |0.91|2374260
H0_18|2515083|22131|0.88|2492952
H0_19|2269381|18001|0.79|2251380
H0_20|2165665|18578|0.86|2147087
SL_01|1401188|17853|1.27|1383335
SL_02|1275971|17011|1.33|1258960
SL_03|1657522|22091|1.33|1635431
SL_04| 1232465|17744|1.44| 1214721
SL_05|1469997|19951|1.36| 1450046
SL_06|1007734|13759|1.37|993975
SL_07|1764706|24481 |1.39|1740225 
SL_08|958682|13589|1.42|945093
SL_09|1722250|22355|1.30|1699895
SL_10|1485114|20830|1.40|1464284
SL_11|1286600|11075|0.86| 1275525
SL_12|1493771|12237|0.82|1481534
SL_13|864739|7489|0.87|857250
SL_14| 2056371|16921|0.82|2039450
SL_15|749087|6223|0.83|742864
SL_16|1740195|15608|0.90|1724587
SL_17|2015396|17753|0.88|1997643
SL_18|1338705|11886|0.89|1326819
SL_19|1214306|10543|0.87|1203763


Used pyRAD on the GDCserver: 

```
module load pyRAD/3.0.6

pyrad -n
```
 
modify the params.txt file as in the pipeline

Run pyrad in screen. Started at 23:33 - 12:00 (21 Aug 2015). ~12.5hrs
```
screen -S pyradSE -L

pyrad -p params.txt -s 234567
```

The average depth for this run (due to low coverage in the HiSeq lanes) = 2-3
Changed the minDepth to 2, and the min number of indivs per locus to 2, for the test run. 

Remember to delete the .loci and the .stats files to rewrite the s7 output! 

**Interesting: The polyfreq of the SE pops looks really similar to that of the Swiss populations! A bit unexpected. 

###11 Sept 2015

##Task 2: pyRAD with larger CH data set

called pyRADlarge

consists of 784 indivs. 

###insert table here. Tab pyrad-largerun in excel file SummaryH1-28_20150714

I demultiplexed and trimmed adapter on fgcz server & partially on the gdc server

I will run pyrad on the gdc server in forlder /gdc_home4/alexjvr/pyRADlarge


```
module load pyRAD/3.0.6

pyrad -n
```

```
==** parameter inputs for pyRAD version 3.0.6  **======================== affected step ==
./                        ## 1. Working directory                                 (all)

./*.fastq.gz              ## 2. Loc. of non-demultiplexed files (if not line 16)  (s1)

./*.barcodes              ## 3. Loc. of barcode file (if not line 16)             (s1)

vsearch                   ## 4. command (or path) to call vsearch (or usearch)    (s3,s6)

muscle                    ## 5. command (or path) to call muscle                  (s3,s7)

TGCAG                     ## 6. Restriction overhang (e.g., C|TGCAG -> TGCAG)     (s1,s2)

20                         ## 7. N processors (parallel)                           (all)

6                         ## 8. Mindepth: min coverage for a cluster              (s4,s5)

4                         ## 9. NQual: max # sites with qual < 20 (or see line 20)(s2)

.94                       ## 10. Wclust: clustering threshold as a decimal        (s3,s6)

ddrad                       ## 11. Datatype: rad,gbs,pairgbs,pairddrad,(others:see docs)(all)

350                         ## 12. MinCov: min samples in a final locus             (s7)

800                         ## 13. MaxSH: max inds with shared hetero site          (s7)

CHc94d6min350                 ## 14. Prefix name for final output (no spaces)         (s7)

==== optional params below this line ===================================  affected step ==

                       ## 15.opt.: select subset (prefix* only selector)            (s2-s7)

                       ## 16.opt.: add-on (outgroup) taxa (list or prefix*)         (s6,s7)

                       ## 17.opt.: exclude taxa (list or prefix*)                   (s7)

/gdc_home4/alexjvr/pyRADlarge/samples/                       ## 18.opt.: loc. of de-multiplexed data                      (s2)

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

             *          ## 30.opt.: output formats: p,n,a,s,v,u,t,m,k,g,* (see docs) (s7)

                       ## 31.opt.: maj. base call at depth>x<mindepth (def.x=mindepth) (s5)

            50           ## 32.opt.: keep trimmed reads (def=0). Enter min length.    (s2)

                       ## 33.opt.: max stack size (int), def= max(500,mean+2*SD)    (s3)

                       ## 34.opt.: minDerep: exclude dereps with <= N copies, def=1 (s3)

                       ## 35.opt.: use hierarchical clustering (def.=0, 1=yes)      (s6)

                       ## 36.opt.: repeat masking (def.=1='dust' method, 0=no)      (s3,s6)

                       ## 37.opt.: vsearch max threads per job (def.=6; see docs)   (s3,s6)

==== optional: list group/clade assignments below this line (see docs) ==================
```

```
screen -S pyRADlarge -L

pyrad -p params.txt -s 234567
```

Run started: 12 Sept 2015 @ 16:30
