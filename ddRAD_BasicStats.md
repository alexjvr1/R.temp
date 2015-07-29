#Basic Statistics



R code for checking whether Haplotype diversity (or other measure) is corrolated wiht latitude or altitude
```
CH <- Table1_elev_CH_20150727

plot(CH$Long., CH$Haplotype.diversity..Hd., main="Elevation ~ haplotype diversity")

```

The input I used was a modification of Table1 from the mtDNA dataset
![alt_txt][input]
[input]:https://cloud.githubusercontent.com/assets/12142475/8954946/3819df28-35e9-11e5-8e6a-ae9fe3fe1941.png

In this case there is no correlation

![alt_txt][input]
[input]:https://cloud.githubusercontent.com/assets/12142475/8954956/556fffd0-35e9-11e5-8c5e-c32f458597ad.png
