#Landscape Genetics MS (Aim MolEcol)

##Introduction 

- The distribution of genetic variation across a species range is determined by many factors
- But one of the most important is dispersal and functional connectivity between populations. 
- The distribution of genetic diversity has an affect on population fitness (ref), range dynamics (ref), and adaptive potential (ref). 
- Understanding factors that influence dispersal will give us insight into the distribution of species (and genetic variation) across their geographically heterogeneous landscape. 


- Lots of connectivity/landscape genetics data available across the flat areas, but we still don't understand connectivity across elevation. 
- Studies of dispersal and connectivity across elevation is often challenging, and thus less studied, for two reasons 1) species often don't occur across a large elevational gradient, 2) this often includes the range edge, thus connectivity dynamics should take into account habitat quality
- But it is interesting because it helps us identify factors in a complex (and often more natural) landscape. 


- For amphibians, much work on lowland landscape genetics has identified open fields/agriculture, and large roads, and dense urban areas as barriers to movement. Forest cover is positively associated with movement. However, it is unclear how these findings translate to the elevational gradient. 


Why is this important?

- Variable connectivity across different gradients in the alps. 
- Really high gene flow across elevation in Scotland. 

Other consideration is that high elevation = range edge. So we have to consider the meta-population dynamics. 
- habitat quality (as measured by wetland/pond depth & ice-free period) affects colonisation

###Final paragraph
Aim:
To determine which landscape and evironmental characteristics affect connectivity across elevation and how this determines the distribution of genetic diversity.

1. Is functional connectivity different across different elevational gradients in the alps? 

2. Topographic complexity are the main factors resulting in resistence in movement across elevation in the alps. (explains high gene flow in Scotland)

2. Breeding site availability/quality determines the range edge.

3. Are regional patterns of connectivity indicative of the distribution of broad scale (phylogeographic-level) genetic diversity? 


##M&M

###Sampling & Genetic data

Sampling was conducted from March-July 2013. In total samples were collected from 44 sites across Switzerland; (1) 30 from the east, and (2) 14 from the south-west (Fig. 1, Table S1). These sites belong to the northern (1) and southern (2) Swiss mitochondrial lineages, as identified in Jasnen van Rensburg et al. xxx. A population was considered a breeding site where ponds were close together (xx distance??), or the whole area covered by a wetland. Euclidean distances between sites ranged from xx - xx overall (xx region 1; xx region 2). Sampling involved collecting eggs from 10-20 clutches, thus reducing the chances of collecting related individuals. Eggs were hatched and individuals were raised until Gossner stage xx before euthenising and preserving them in absolute ethanol. In total xx individuals were sampled. 

*DNA extraction and library preparation*
DNA was extracted from tadpole tails using Qiagen BioSprint 96 DNA Blood Kit. Genome-wide genetic variation was assessed using single-end double digest Restriction Associated DNA (RAD) sequencing. Library preperation was conducted at the Genomic Diversity Center, Zurich following the protocol of Parchman *et al.* 2012 with modifications as detailed in the Supporting information xxx. Samples were individually labelled, with 48 samples pooled per library. Libraries were sequenced on an Illumina HiSeq 2500 v4 at the Functional Genomics Cetnter, Zurich (FGCZ). 

*Bioinformatics pipeline*
Raw data was demultiplexed using the process_radtags component from the Stacks v xx pipeline (Catchen et al. 2013), and reads from samples that had been included in two sequencing lanes were merged. Samples were filtered for adapter dimer using xxx (ref). The remaining reads were de novo assembled using the pyRAD v3.0.6 pipeline (Eaton et al. xx). Non-stringent filtering thresholds were used during the de novo assembly, while the final SNP data set was filtered by applying different filtering parameters in VCFtools (ref). Filtering parameters in pyRAD: mindepth 5, min missing xx, heterozygosity was set very high..  One alignment threshold is specified for both within and between sample cluster alignments. This value was optimised with a subset of the data representing the geographic diversity of all the samples (Supplementary material xx). 

####This bit is from Stervander et al. 2015. Check and adjust accordingly: 

The data output from pyRAD was then further filtered using VCFtools. We filtered output so that it contained only loci i) in which all samples had been assigned 1-2 alleles, ii) which contained 1-10 SNPs, iii) which comprised =/<12 alleles. The rationale for these filters was to remove any loci in which individuals had been assigned >2 loci and or were hypervariable (but values were chosen arbitrarily after inspecting the data).   


Q1/Compare genetic structure across different elevations in the Alps

Population structure and demographic history has been previously described.. ?? Publish this MS first??

- Average genetic diversity

- genetic diversity as a fxn of elevation

- Genetic distance across elevation 
           - IBD across elevation 

- Relatedness estimate within populations, and across elevation, compared to random combinations (Queller & Goodnight??)




Q2/ Which landscape features predict functional connectivity?

Test the following: 

- Null hypothesis is IBD: Fst/1-Fst


Method:

1. ResistanceGA (Peterson et al. 2014)

2. Gravity model as in Murphy et al. 2010


Q3/ Does breeding site quality determine range edge?

- Measured breeding site quality by:
           - predicting the presence of predators (depth)

Method

- Gravity model (Murphy et al. 2010)





##Results





##Discussion





##Conclusion




##Acknowledgements

Janine Bolliger for GIS support and providing landscape data, WSL for access to Swiss land cover and evironmental data. 






Papers: 

1. Trumbo et al. 2013 
- investigate landscape genetics at larger scale in Cope's Giant Salamander. And find that landscape resistance is different in different parts of the range. 

2. Decout *et al.* 2012
3. Murphy et al. 2010 Landscape genetics of high mountain frog metapopulations. 
 

3.
