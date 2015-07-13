##*R.temp* Phylogeography



*Aim*
Three major factors determine the current distribution of amphibian species: 1)biogeographic history, 2)fine scale connectivity between populations (landscape genetics), 3)and survival in their current environments (adaptation to local environments/landscape genomics). 
To obtain a holistic view of the enviro-evolutionary processes that determine the persistance 

But to study the adaptibility of species across environmental gradients, we should understand the underlying distribution of genetic diversity across the gradient. 
Here we implement a comparative phylogeographic approach to describe the colonisation history of Rana temporaria across an elevational and latitudinal gradient corresponding to the same change in environmental parameters. 

Our focal species is Rana temporaria, one of the most wide-spread anurans. 
This makes them an ideal candidate for studying adaptation 
We know that there is a deep divergence between the Eastern and Western R. temporaria clades, which is likely to have occurred during the onset of the current cycle of climatic oscillations (0.7mya) (Palo et al. 2004). This is the same time that the R. temporaria radiation occurred (Veith et al. 2003?? is this the right ref?).
The Western clade is characterised by high genetic diversity and population structure (Palo et al. 2004). It is suggested that much of this is due to cryptic refugia in central Europe during the last glacial maximum (ref). 
In contrast, the Eastern clade is depauperate of genetic diversity. 

The aim of this study is to study the distribution of genetic variation in Rana temporaria at a fine-scale across Sweden and Switzerland, in order to understand the processes that have resulted in the current spatial genetic variation across the two temperature gradients. 

1. Reconstruct the fine-scale re-colonisation history of Switzerland and Sweden
2. Characterise geographic distribution of genetic diversity across these similar environmental gradients. 





Drew a map of the sampling locations using R & Adobe Illustrator. 

- SE samples from N. Rodriguez & A. Laurila
- Could draw the map in gray i.s.o colour
- for code see on GitHub: alexjvr1/R.temp/SamplingMap_Europe.md


Excel sheet of sample data: 

SampledLocalities_coords_20150615.xlsx (up to date as of 17 June 2015)



##Generating the data
###Lab work


###Sequence alignments
All sequences were aligned and cleaned using BioEdit and CLCBio. 
The final dataset comprises:
cytb:
- Sequences from Vences *et al.* 2013 for comparison: KC799522.1 - KC800122.1
- Outgroups: 
    - *R. pyrenaica*: KC799521.1
    - *R.t.parvipalmata*: 
    - *R.a.arvalis*: KC800123
    - *R.iberica*: KC799485
    - *R.italica*: KC799493

- 448bp cytb for 173 CH (44 pops) and 34 SE (9 pops) samples


*Final cytb Datasets*

- 448bp cytb for 169 CH (44 pops), 32 SE (9 pops)
- 331bp cytb for 169 CH (44 pops), 32 SE (9 pops), 601 Vences et al. (106 pops)
- Haplotypes: 331 bp, 59 haplotypes (concatenated with Vences et al. data)

For the Haplotype data: 

Vences et al. report 47 haplotypes for the cytb data. However, with the data available on NCBI and supplementary table S3, I can assign only 40 of these to haplotypes, and there is one haplotype (ALC02, XAL01 - Spain) for which the samples are not reported in table S3. 

My own data will be named H48-59. 

The naming assignments are kept in: cytb+COX1_HaplotypeAssignments_20150618.xlsx






COX1:
- Sequences from Vences *et al.* 2013 for comparison: KC977228.1-50.1 from NCBI
- Outgroups: 
    - R. pyrenica: KC977251.1 (Vences *et al.* 2013)
    - R.t.parvipalmata - none available
    - R.a.arvalis: JN871596.1 
    - R.iberica/italica - none available 

*Final COX1 Datasets*

- 628bp COX1 for 92 CHN and 59 CHS (53 pops) 33 SE (9 pops) samples  
- 628bp COX1 as above + sequences from Vences et al. 2013
- 628bp COX1 CH + SE + Vences + outgroups (R.arvalis + R.pyrenica)

All data are stored on my HP laptop, and can be found here: 
- C:/hp_1_PhD_AJvR_20130101/1_Thesis/Chapter2_Phylogeography/Analyses/mtDNA_Datasets (Datasets: see next section)
- C:/hp_1_PhD_AJvR_20130101/1_Thesis/Chapter2_Phylogeography/Analyses/1_* (raw data)


*Concatenated Datasets*

448bp cytb + 628bp COX1. Both cytb and COX1 are in frame, but 2 bases need to be added in between to keep COX1 in frame. 

- 1076bp cytb+COX1 CHN (90), CHS (56), + SE (27)
- 1076bp as above + R.arvalis + R.pyrenica

##Exploring the data

I am comparing my data with that available in Vences *et al.* 2014. He has cytb and COX1 sequences available that overlap with mine, and the data provides a Europe-wide context within which to explore my data. 


1. Haplotype networks ####(Still need the figures)
2. Table 1
    - I am generating a table (Table 1 in MS) with genetic diversity and basic statistics
3. AMOVA
4. Phylogenetic Trees: ML, MrBayes
5. Basic statistics (test for selection, Fst)
6. IBD
7. Demographic analyses: BSP, Mismatch distribution
8. BEAST: dating the split

###Haplotype networks

1. Export Phylip format from dnaSP
2. Import into TCS
3. Draw haplotype network with 100step connection limit
4. Update the mtDNA data file to show which individuals have which haplotypes
5. Take note of the number of individuals in each haplotype
6. Draw haplotype networks in Adobe Illustrator

And secondly map the proportion of CHN and CHS on a map of Switzerland
See the PieChartsonMap_R.md on my GitHub page for a tutorial


###Table 1
This table is for comparison with Stefani *et al.* 2012
This should contain the number & names of haplotypes in each population for each gene, the Haplotype diversity, and nucleotide diversity

I found a N/S split across the alps in my data, so I am naming the Haplotypes H1 (South) and H2 (North). 
In each case .1 is the common haplotype, and .2...n are the derived haplotypes (as determined by the haplotype network)

Samples that fall into N & S depending on whether they were cytb or COX1 sequences will be removed. They need to be resequenced, but I don't have time before the data analysis: 

- arce 02
- fada 02
- fuor 01

Haplotype names and assignment for COX1 & cytb
```
1. COX1 sequences (SE+CH) with those from Vences *et al.* 2014
2. Convert to nexus format using MEGA6
3. Import into dnaSP and run: Generate -> Haplotype data file
    This will provide a breakdown of all the haplotypes and which individuals they are assigned to
4. Work out which haplotypes are H1 and H2 and put the information in Table 1

Do the same for cytb
```
**I have found 2 SE COX1 sequences that seem to have been incorrectly named during sequencing.
    - nord12: I have one sequence with the common SE haplotype and one with a CH haplotype. I will use the SE haplotype and remove the CH nord12 sample
    - mjos07: same haplotype as gruu01, so a derived CH-N haplotype. This is probably incorrect, so I will remove mjos07 from further analyses. 
    
    

***For the cytb sequences:

    - I've sequenced ~500bp for my samples, but this overlaps with only ~330bp of Vences *et al.* sequences. So there are 2 data sets. 
    1.1 cytb CH (173indivs, 480bp): 5_cytb_CH_20150215.meg
    1.2 cytb CH haplotypes only: 5_cytb_CH_Haplotypes_20150215.nex
    2.1 cytb CH + SE (205 (172+33)indivs, 448bp): cytb_CH_SE_448_20150623.nex
    2.2


***The cytb sequence of *stir02* falls in with one of the major SE haplogroups. I think this is probably a mistake, so stir02 will be removed from all the rest of the analyses. (from the cytb dataset). 
***nord18 is really divergent to the other sequences. I think there is probably some mistake. I'm removing the sequence from all further analyses. 

Draw a haplotype network of the combined datasets using TCS
```
Input for TCS: 
Need Nexus file which can be obtained from dnaSP. Export as Nexus (Options Nexus old version, CR+LF(PC))
    - Have to manually delete all the returns from the file. I haven't yet figured out how to get the correct file format. 
    - TCS is very finicky with it's input format. 
    
Final file for COXI (full dataset): 11_COX1_CH_SE_Vences_full_20150619.nex (this already has the nord12 & mjos07 samples deleted, and has samples after rusc06 - samples after rusc06 were omitted in many of the COX1 files)
And the graph is saved under the same name. 
#I still have to draw the map and the haplotype network for this figure.  (TCS figure saved as 11_...figure.nex.graph

Final file for the cytb (full dataset): 
#I need to redraw the haplotype network for this and adapt the figure using the map from Vences et al. 2014

```

Now that I've identified the samples assigned to CH-N and CH-S, I can complete Table 1
```
Import the Nexus file into dnaSP
To assign individuals to specific groups: 
    Data -> Define Sequence sets
        *And now you can select the individuals in the different groups
        
To calculate haplotype diversity & nucleotide diversity (Pi)
    -Calculate polymorphism data (Overview -> Polymorphism data)
    
    - I'm using the haplotype diversity (with SD)
    - And nucleotide diversity (with SD)
```

##AMOVA

AMOVA determines where the most genetic diversity occurs: within or between groups. 

This analysis is run in Arlequin

*I had some problems with Arlequin: Make sure the WinArl folder is located in the C drive on Windows (I think it doesn't like spaces in folder names). And make sure to save the settings after specifying the analyses (arl_run.ars).*

####*Input*####

Arlequin input files can be exported from dnaSP.

1. Export the .arp and the .hap file from dnaSP
2. copy the haplotypes from the .hap file
3. Below [[Haplotype Definition]] type: HaplList={
4. Paste the haplotypes
5. Below the haplotypes, close the bracket }
6. Save as .arp


####*Run AMOVA*####

1. Load the data into the Arlequin GUI
2. Make sure to specify the text editor
3. Assign groups in Structure Editor
4. Under the Settings tab, select AMOVA, 1000 permutations
5. Save
6. Start
7. Output is in a new folder with the name of the input file (.xml file) and can be opened with textpad or similar


####*cytb*####

For cytb I want to know

1. Whether most of the CH variation is within populations or between CHN and CHS
2. Whether most of the overall variation is within populations or between SE and CH

For (1):

1. Define all the populations in dnaSP as N or S. For this analysis I made the a priori decision based on the haplotype network. So if a population has 1CHN and 3CHS, there were two groups called in dnaSP with 1 and 3 samples respectively. 

2. Results were as follows: 

Source of Variation | d.f. | Sum of Squares | Variance Component | Percentage of Variation
:---:|:---:|:---:|:---:|:---:
Among Groups |1|259.399|3.16511 Va| 88.35
Among Pops Within Groups |52|45.113|0.21287 Vb| 5.94
Within Pops |115|23.500|0.20435 Vc| 5.70
Total|168|328.012|3.58233

Significance tests:

All 0.00000

For (2): 

1. Define all the populations in dnaSP for CH (i.e. don't split CHN and CHS). 
2. Results were as follows: 

Source of Variation | d.f. | Sum of Squares | Variance Component | Percentage of Variation
:---:|:---:|:---:|:---:|:---:
Among Groups |1|249.977|4.68678 Va| 72.60 
Among Pops Within Groups |49|245.252|1.16675 Vb|18.47
Within Pops |143|80.750|0.56469 Vc|8.94
Total|193|575.979|6.31821

Significance tests:

All 0.00000

####*COX1*####

For COXI I want to know the same as for cytb:

1. Whether most of the CH variation is within populations or between CHN and CHS
2. Whether most of the overall variation is within populations or between SE and CH

For (1):

1. Define all the populations in dnaSP as N or S. For this analysis I made the a priori decision based on the haplotype network. So if a population has 1CHN and 3CHS, there were two groups called in dnaSP with 1 and 3 samples respectively. 

2. Results were as follows: 

Source of Variation | d.f. | Sum of Squares | Variance Component | Percentage of Variation
:---:|:---:|:---:|:---:|:---:
Among Groups |1|192.935|2.74657 Va| 88.84
Among Pops Within Groups |49|37.863|0.21996 Vb|7.12
Within Pops |100|12.5|0.12500 Vc|4.04
Total|150|243.298|3.09153

Significance tests:

All 0.00000

For (2): 

1. Define all the populations in dnaSP for CH (i.e. don't split CHN and CHS). 
2. Results were as follows: 

Source of Variation | d.f. | Sum of Squares | Variance Component | Percentage of Variation
:---:|:---:|:---:|:---:|:---:
Among Groups |1|431.814|7.88329 Va|85.12 
Among Pops Within Groups |58|231.018|1.27141 Vb|13.73
Within Pops |124|13.250|0.10685 Vc|1.15
Total|183|676.082|9.26155

Significance tests:

All 0.00000


####*Concatenated Dataset*####

Same questions as above:

1. Whether most of the CH variation is within populations or between CHN and CHS
2. Whether most of the overall variation is within populations or between SE and CH

For (1):

1. Define all the populations in dnaSP as N or S. For this analysis I made the a priori decision based on the haplotype network. So if a population has 1CHN and 3CHS, there were two groups called in dnaSP with 1 and 3 samples respectively. 

2. Results were as follows: 

Source of Variation | d.f. | Sum of Squares | Variance Component | Percentage of Variation
:---:|:---:|:---:|:---:|:---:
Among Groups |1|426.010|6.15297 Va| 91.18
Among Pops Within Groups |49|53.178|0.26588 Vb|3.94
Within Pops |95|31.250|0.32895 Vc|4.87
Total   |145|510.438|6.74779

Significance tests:

All 0.00000

For (2): 

1. Define all the populations in dnaSP for CH (i.e. don't split CHN and CHS). 
2. Results were as follows: 

Source of Variation | d.f. | Sum of Squares | Variance Component | Percentage of Variation
:---:|:---:|:---:|:---:|:---:
Among Groups |1|577.971|12.47750 Va|80.14
Among Pops Within Groups |58|482.161|2.79242 Vb|17.94
Within Pops |113|33.833|0.29941 Vc|1.92
Total  |172|1093.965|15.56933

Significance tests:

All 0.00000

##Demographic Analyes

1. Mismatch distribution: Arlequin
     - See AMOVA section above for details to load data file into Arlequin
     - Under settings, choose Mismatch distribution -> Estimate parameters of demographic expansion + 1000 bootstrap replicates

    

2. Bayesian Skyline Plot: BEAST


####*cytb*

*Mismatch Distribution*

Sudden expansion model tested

Results in:  CHAllPops.xml

#####*CHall*

Test of goodness-of-fit | Values
:---:|:---:
Sum of Squared Deviations | 0.12740512
P(Sim. Ssd >= Obs. Ssd) | 0.004000
Harpending's Raggedness index | 0.22447845
P(Sim. Rag. >= Obs. Rag.) | 0.00630000

#####*CHS*

Test of goodness-of-fit | Values
:---:|:---:
Sum of Squared Deviations | 0.15219882
P(Sim. Ssd >= Obs. Ssd) | 0.02740000
Harpending's Raggedness index | 0.54809296
P(Sim. Rag. >= Obs. Rag.) | 0.00850000


#####*CHN*

Test of goodness-of-fit | Values
:---:|:---:
Sum of Squared Deviations | 0.00463243
P(Sim. Ssd >= Obs. Ssd) | 0.02050000
Harpending's Raggedness index | 0.16260941
P(Sim. Rag. >= Obs. Rag.) | 0.730000

#####*SEall*

Error: Least-square procedure to fit expected and observed mismatch distribution did not converge after 2000 steps




*Bayesian Skyline Plot*



####*COX1

*Mismatch Distribution*

Parameters estimated under the sudden expansion model

Results in COX1_MM

#####*CHall*



Test of goodness-of-fit | Values
:---:|:---:
Sum of Squared Deviations | 0.13675574
P(Sim. Ssd >= Obs. Ssd) | 0.02780000
Harpending's Raggedness index | 0.22561598
P(Sim. Rag. >= Obs. Rag.) | 0.08760000



#####*CHS*

Results in COX1_MM

Test of goodness-of-fit | Values
:---:|:---:
Sum of Squared Deviations | 0.00005236
P(Sim. Ssd >= Obs. Ssd) | 0.9999
Harpending's Raggedness index | 0.09079252
P(Sim. Rag. >= Obs. Rag.) | 0.03300000


#####*CHN*

Error: Least-square procedure to fit expected and observed mismatch distribution did not converge after 2000 steps

#####*SEall*

Error: Least-square procedure to fit expected and observed mismatch distribution did not converge after 2000 steps





*BSP*


####*Concatenated Dataset*

*Mismatch Distribution*

Test for sudden expansion

Results in WinArl35/Concat_allPopsNamed_test1.res folder

#####*CHS*

Test of goodness-of-fit | Values
:---:|:---:
Sum of Squared Deviations | 0.03382908
P(Sim. Ssd >= Obs. Ssd) | 0.1940000
Harpending's Raggedness index | 0.10279010
P(Sim. Rag. >= Obs. Rag.) | 0.31200000


#####*CHall*

Test of goodness-of-fit | Values
:---:|:---:
Sum of Squared Deviations | 0.09724376
P(Sim. Ssd >= Obs. Ssd) | 0.003
Harpending's Raggedness index | 0.08403806
P(Sim. Rag. >= Obs. Rag.) | 0.044000

#####*CHN*

Error: Least-square procedure to fit expected and observed mismatch distribution did not converge after 2000 steps

#####*SEall*

Error: Least-square procedure to fit expected and observed mismatch distribution did not converge after 2000 steps



*BSP*



##Phylogenetics

1. Model Selection using jModelTest2 on Cipres
2. ML tree using RXML
3. Bayesian tree using MrBayes

####*cytb*

Generate a dataset with a representative each of the haplotypes from Vences et al, and from my own data.
As outgroups, I'm using R.italica, R.iberica, R.arvalis, R.pyrenaica sequences from Vences et al.

I aligned 60 sequences (57 haplotypes + 2CH and 1SE representatives of the Vences haplotypes), and cut them down to 331bp alignment. 


*jModelTest*

- Read the fasta alignment file into jModelTest2 on Cipres online platform
- Test 88 models against each other

The most likely model using AIC for cytb with outgroups: 

Model = TIM3+I+G

partition = 0.12032

-lnL = 1369.6723

K = 126

freqA = 0.2435

freqC = 0.3402

freqG = 0.1084

freqT = 0.3079

R(a) [AC] = 0.3124 

R(a) [AG] = 34.8455

R(a) [AT] = 1.0000

R(a) [CG] = 0.3124

R(a) [CT] = 15.4882

R(a) [GT] = 1.0000

p-inv = 0.4510

gamma shape = 0.7200



####*COX1*

Generate a dataset with a representative each of the haplotypes from Vences et al, and from my own data.
As outgroups, I'm using R.arvalis & R.pyrenica sequences from NCBI (JN871596.1 + ZUR02 from Vences et al. 2013, respectively). 

I aligned xx sequences (xx haplotypes), and cut them down to xxbp alignment. 


*jModelTest*

- Read the fasta alignment file into jModelTest2 on Cipres online platform
- Test 88 models against each other

The most likely model using AIC for *COX1* with outgroups: 


####*Concatenated Dataset*

Generate a dataset with a representative each of the haplotypes from Vences et al. 2013, and from my own data.
As outgroups, I'm using R.arvalis & R.pyrenica sequences from NCBI (JN871596.1 + ZUR02 from Vences et al. 2013, respectively). 

I aligned xx sequences (xx haplotypes), and cut them down to xxbp alignment. 


*jModelTest*

- Read the fasta alignment file into jModelTest2 on Cipres online platform
- Test 88 models against each other

The most likely model using AIC for the Concated Dataset with outgroups: 






##Date Divergence

1. Beast
  - Using % calibration from literature
  - Using a fossil calibration
  

For tree prior: 
- Populations -> coalescent
- across genera -> Birth-Death model

For time calibration
- When there's a biogeographical calibration point, use a normal distribution 

- possible calibration points for me:
    - Biogeographical: E/W lineage split after last ice age
    - Fossil? R. arvalis and R. temporaria split (Veith et al. 2003)
    - 



##Basic Statistics

Is the region under selection?

1. Define coding region
```
I aligned my cyt b sequence to an available R.t.parvipalmata from NCBI and found the ORF by comparing the region they suggest is an ORF, and translating the fragment using web.expasy.org/cgi-bin/translate/dna_aa
For COX1, I just translated with the website to find the most likely ORF

For cytb: ORF starts on position 1. 
COX1: ORF starts on position 1.
```

To check for selection I'm doing a codon-based test of neutrality for analyses averaging over all sequence pairs within each group. 
```
1. Load data into Mega6
2. Define groups (Data -> Select Taxa and Groups)
3. Selection test (Selection -> Codon-based Z-test of selection)
              Scope: Average within groups
              Test Hypothesis: Neutrality (HA: dN =/= dS)
              Variance estimation method: Bootstrap, 1000 reps
              Model: Nei-Gojobori method
              Gaps/missing data: Partial deletion, 95% cutoff
```


Results & Caption: cytb

group | n | positions | stat | P
:---: | :---: | :---: | :---: | :---:
CHN | 69 | 149 | -1.591 | 0.114
CHS | 100 | 149 | -2.932 | *0.004* 
CHall | 169 | 149 | -3.197 | *0.002* 
SE |32 | 149 |-1.465 | 0.146


COX1

group | n | positions | stat | P
:---: | :---: | :---: | :---: | :---:
CHN | 92 |209  |-2.550 |*0.012* 
CHS |59 |209 |-2.267 |*0.025*  
CHall | 151 | 209 |-2.882  |*0.005* 
SE | 33|209 |-1.014|0.312 


```
The probability of rejecting the null hypothesis of strict neutrality (dN = dS) is shown. P < 0.05 are considered significant. The test statistic dN-dS is shown in the stat column. dS and dN are the numbers of synonymous and nonsynonymous substitutions per site, respectively. The variance of the difference is computed using the bootstrap method (1000 replicates). Analyses were conducted using the Nei-Gojobori method (Jukes-Cantor). All positions with less than 95% site coverage were eliminated. That is, fewer than 5% alignment gaps, missing data, and ambiguous bases were allowed at any position. Analyses were conducted in MEGA6. 
```

Ref for MEGA6: 

Tamura K, Stecher G, Peterson D, Filipski A, Kumar S (2013) MEGA6: Molecular Evolutionary Genetics Analysis version 6.0. *Molecular Biology and Evolution*, 30, 2725-2729.

