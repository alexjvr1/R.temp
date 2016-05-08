#Optimisation of pyRAD thresholds: 

1. Clustering threshold

  Checks: cluster vs nucleotide diversity, vs 
  
2. MinDP

pyRAD uses the ML method from Lynch et al. 2008 to jointly estimate nucleotide heterozygosity and error for every base across a stack. 

This is estimated per stack within an individual to call homozygotes/ heterozygotes. 

The method seems robust from ~6-8x coverage, unless error rates are really high, or nucleotide diversity is very low. 


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

**These ran on GDCsrv. and vcftools/bcftools will be run on the mac. 
/Users/alexjvr/2016RADAnalysis/pyRADopt/input

Plot: 

1. average depth per individual vs nucleotide diversity

```


```


2. Average depth per individual vs Heterozygosity

```


```
3. Average depth per site vs nucleotide diversity
```
vcftools --vcf CHDepthtest.MinDP10.vcf --site-mean-depth --recode-INFO-all  ##this information is not in the vcf file! I guess it gets removed by pyRAD, which only outputs the SNP information... 

vcftools --vcf CHDepthtest.MinDP10.vcf --site-pi --recode-INFO-all
```

Write column 3 to a new file called freq.pi. Move this to gdcsrv to draw a histogram in bash: 

/gdc_home4/alexjvr/DepthTest/CH.plots


```
cat out.sites.pi | awk '{print $3}' > freq.pi

scp freq.pi alexjvr@gdcsrv1.ethz.ch:/gdc_home4/alexjvr/DepthTest/CH.plots/


gnuplot << \EOF 
set terminal dumb size 120, 30
set autoscale 
unset label
set title "Histogram of nucleotide diversity per locus"
set ylabel "Number of Occurrences"
set xlabel "nucleotide diversity"
binwidth=0.01
bin(x,width)=width*floor(x/width) + binwidth/2.0
plot 'freq.pi' using (bin( $1,binwidth)):(1.0) smooth freq with boxes
pause -1
EOF
```

MinDP10 

![alt_txt][CH.DP10_site-pi]
[CH.DP10_site-pi]:https://cloud.githubusercontent.com/assets/12142475/15096083/694e993c-149e-11e6-8934-cf5ffb21feb1.png


MinDP15

![alt_txt][CH.DP15_site-pi]
[CH.DP15_site-pi]:https://cloud.githubusercontent.com/assets/12142475/15096085/6f2191d4-149e-11e6-9170-ab624d220d50.png




To check how much the depth filter makes a difference in pyRAD: 

compare avg depth per site vs nucleotide diversity for n10 dataset, filtered for n15 coverage, to pyRAD-ru n15 coverage

```

```


4. Average depth per site vs Heterozygosity
```

```

5. Ho vs He at Depth10 vs Depth 15
```

```
 






###BV

49 individuals from demultiplexedBV2and3

I've transferred these to Euler & will run MinDP10, MinDP15, MinDP20 for these



##Submitting jobs to Euler

```
ssh alexjvr@gdcsrv1.ethz.ch
jk.W3.mm
ssh alexjvr@euler.ethz.ch
W465.Smq

cp pyrad.run.cmds /cluster/home/alexjvr/ /cluster/scratch/alexjvr/*file*

#adjust parameters in cmds file

  - time for each run
  
  - # out any commands you don't want run

nano params.txt

#adjust any parameters

./pyrad.run.cmds
```


##See Stefan's email explaining all of this - in the flagged folder in my inbox. 

```
Hi Alex

Below you find the how-to for running pyrad on Euler. It includes the installation of the "package", what to copy where and information on how to actually start a job. 
Attached you find the files that are mentioned below. 
First you have to copy these files to our servers, e.g. gdcsrv1

1) on gdcsrv1, copy the pyrad package to euler (please note the colon at the end)
    scp pyrad-3.0.63.tar  alexjvr@euler.ethz.ch:
   do the same for the pyrad.run.cmds
    scp pyrad.run.cmds  alexjvr@euler.ethz.ch:
2) login to euler
3) create a bin directory (unless already present)
    mkdir bin
4) now move the package to bin and change into it
    mv pyrad-3.0.63.tar bin/
    cd bin/
6) extract the package
    tar xvf pyrad-3.0.63.tar
7) change to the home again
    cd ..
8) Use the command-line editor nano to open the file .bashrc
    nano .bashrc
   now enter the following line after # User specific aliases and functions:
    export PATH=/cluster/home/alexjvr/bin/pyrad-3.0.63/pyrad/:$PATH
   To save and exit: 
   press control-o, then hit return, then control-x
9) now test if pyrad is available
    source .bashrc
    which pyrad 
10) now move to your scratch space and create a run directory and change into it
     cd /cluster/scratch/alexjvr/
     mkdir pyrad_1
     cd pyrad_1 
11) copy the pyrad.run.cmds from your home to the current location
     cp ~/pyrad.run.cmds . 
12) adjust the settings in the pyrad.run.cmds. Especially important is the 
    memory and the run time (wall-clock-time).
     There is for example: 
     -n the number of CPUs. You probably always want 24
     -W for wall-clock-time. The job will run for at 
     most that long (-> will be put into scheduler queue according to this)
     adjust this according to my estimates. Anything longer than 120 hours goes
     into the 30d queue and might then wait for several hours before starting. 
     -R "span[hosts=1]", makes sure it runs on one node.
     -R "rusage[mem=500]", for reserving memory. This is per CPU, so in our 
     case multiplied by 24. If you want all memory on a 256GB node, then put
     mem=10500
    Usually, I make sure to give a job enough resources, because otherwise it 
    might get killed by the scheduler prematurely. On the other hand do not 
    request too much, because then it might slip into a "higher" queue.
     -J "step2", gives the command a name, which can be read by the next 
     -w "ended(step2)", wait until command "step2" is done
     And finally of course the pyrad commands, e.g.:pyrad -p params.txt -s 2
    Note: you can out-comment a command with #, like I did with step 1,6,7
13) make the pyrad.run.cmds file executable:
     chmod 755 pyrad.run.cmds
14) adjust your params.txt file, especially the location of the input files. 
15) start the pyrad job:
    ./pyrad.run.cmds 
    This should print a couple of 
        Generic job.
        Job <11204117> is submitted to queue ...
16) now you can verify that the jobs are in the queue and/or running:
     bjobs
17) once a subjob is done, or an error occurred, you get an lsf.o.... file 
    In there you will find info about the jobs resource usage and status. 

OK, that was a long how-to, I hope all is clear and will work. Otherwise, let me
know and I will try to help. Or you could come to the GDC and we could start the 
first jobs together. 
```


