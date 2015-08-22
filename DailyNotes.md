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

