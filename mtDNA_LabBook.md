##*R.temp* Phylogeography

*Aim*

1. Reconstruct colonisation history of Switzerland
2. Characterise geographic distribution of genetic diversity


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


Haplotype names and assignment for COX1

1. COX1 sequences (SE+CH) with those from Vences *et al.* 2014
2. 




##Demographic Analyes

1. Mismatch distribution: Arlequin
2. Bayesian Skyline Plot: BEAST



##Date Divergence

1. Beast
  - Using % calibration from literature
  - Using a fossil calibration
  







