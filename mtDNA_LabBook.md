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
- 448bp cytb for 173 CH (44 pops) and 34 SE (9 pops) samples
- 628bp COX for 154 CH (41 pops) 33 SE (9 pops) samples  

All data are stored on my HP laptop, and can be found here: 
- C:/hp_1_PhD_AJvR_20130101/1_Thesis/Chapter2_Phylogeography/Analyses/mtDNA_Datasets (Datasets: see next section)
- C:/hp_1_PhD_AJvR_20130101/1_Thesis/Chapter2_Phylogeography/Analyses/1_* (raw data)



##Exploring the data

I am comparing my data with that available in Vences *et al.* 2014. He has cytb and COX1 sequences available that overlap with mine, and the data provides a Europe-wide context within which to explore my data. 


1. Haplotype networks
2. Table 1
    - I am generating a table (Table 1 in MS) with genetic diversity and basic statistics
3. AMOVA
4. Phylogenetic Trees

###Haplotype networks


###Table 1
This table is for comparison with Stefani *et al.* 2012
This should contain the number & names of haplotypes in each population for each gene, the Haplotype diversity, and nucleotide diversity

I found a N/S split across the alps in my data, so I am naming the Haplotypes H1 (South) and H2 (North). 
In each case .1 is the common haplotype, and .2...n are the derived haplotypes (as determined by the haplotype network)


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


##Demographic Analyes

1. Mismatch distribution: Arlequin
2. Bayesian Skyline Plot: BEAST



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





