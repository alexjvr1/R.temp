#Optimisation of pyRAD thresholds: 

1. Clustering threshold

  Checks: cluster vs nucleotide diversity, vs 
  
2. MinDP

  Checks: nucleotide diversity vs depth, Ho vs He at different depths
  
  
3. Paralog filter??
  
  I'm currently filtering paralogs that are >2 SD from the mean depth. But I have such a high SD in my depth, that this doesn't seem like a very good filter. 
  
  I'm not sure how I can correct for this. 
  
  
##Clustering Threshold

Run pyRAD for a subset of individuals (a good representation of the genetic diversity in the population). 

If samples are overclustered, we expect higher nucleotide diversity & Heterozygosity. 

Underclustering will lead to more loci, but a drop-off in nucleotide diversity. 

Use the output from the stats/Pi_* file to create the plots. 




##Depth

UCLA groups found that the He/Ho tend more towards convergence when a higher minDP is used (i.e. minimum number of reads clustered across individuals.) 

To test whether my data show the same effect I will run minDP 6, 10, 20 initially. 

PyRAD needs to be run from s2. If I subset the samples that have already run (edits/ & clust.94/), it doesn't work. 

###SE

I'm using the 36 pyRADopt individuals. 

minDP 6 

minDP 10 (gdcsrv1)

minDP 20 (gdcsrv2)


###CH

I'm using 55 individuals randomly chosen from the FstQst sample folder. 

I've transferred these to Euler & will run MinDP10, MinDP15, MinDP20 for these

###BV

49 individuals from demultiplexedBV2and3

I've transferred these to Euler & will run MinDP10, MinDP15, MinDP20 for these


