##ddRAD wet lab protocol

ddRAD protocol (caliper size selection) optmised by AlexJvR in 2015

Species: *R. temporaria*

Modified from:
- Alan Brelsford's protocol (based on Parchman *et al.* 2011)
- Claudia Michel's protocol (based on Peterson *et al.* 2012)
- GeCKo protocol from Paulo Franchini (based on Peterson *et al.* 2012) 

All oligo sequences given as 5'-3'

####Adapters:
Adapters can be made up with water or buffer. Buffer is more stable for long-term storage.

*Annealing buffer stock*:

|**Amount**|**Reagent**|
|:---:|:---|
100 mM |Tris HCl, pH 8
500 mM |EDTA
2 M |NaCl

####Adapter P1 = *Eco*RI x 48 *(provided by the GDC)*

|**Nr**|**Sequence**|
|:----:|:--------:|
|1.	|GCATG_EcoRI
2.	|AACCA_EcoRI
3.	|CGATC_EcoRI
4.	|TCGAT_EcoRI
5.	|TGCAT_EcoRI
6.	|CAACC_EcoRI
7.	|GGTTG_EcoRI
8.	|AAGGA_EcoRI
9.	|AGCTA_EcoRI
10.	|ACACA_EcoRI
11.	|AATTA_EcoRI
12.	|ACGGT_EcoRI
13.	|ACTGG_EcoRI
14.	|ACTTC_EcoRI
15.	|ATACG_EcoRI
16.	|ATGAG_EcoRI
17.	|ATTAC_EcoRI
18.	|CATAT_EcoRI
19.	|CGAAT_EcoRI
20.	|CGGCT_EcoRI
21.	|CGGTA_EcoRI
22.	|CGTAC_EcoRI
23.	|CGTCG_EcoRI
24.	|CTGAT_EcoRI
25.	|CTGCG_EcoRI
26.	|CTGTC_EcoRI
27.	|CTTGG_EcoRI
28.	|GACAC_EcoRI
29.	|GAGAT_EcoRI
30.	|GAGTC_EcoRI
31.	|GCCGT_EcoRI
32.	|GCTGA_EcoRI
33.	|GGATA_EcoRI
34.	|GGCCA_EcoRI
35.	|GGCTC_EcoRI
36.	|GTAGT_EcoRI
37.	|GTCCG_EcoRI
38.	|GTCGA_EcoRI
39.	|TACCG_EcoRI
40.	|TACGT_EcoRI
41.	|TAGTA_EcoRI
42.	|TATAC_EcoRI
43.	|TCACG_EcoRI
44.	|TCAGT_EcoRI
45.	|TCCGG_EcoRI
46.	|TCTGC_EcoRI
47.	|TGGAA_EcoRI
48.	|TTACC_EcoRI


####Adapter P2 (from Alan Brelsford's R.temp work):

#####P2.1

*MSe*I-adap1-bar
5' TAAGATCGGAAGAGCACACGTCTGAACTCCAGTCA 3'

#####P2.2

*Mse*I-adap2-bar
5' CTGGAGTTCAGACGTGTGCTCTTCCGATCT 3'

####Illumina PCR primers
#####ILLPCR1

5' A-A-TGATACGGCGACCACCGAGATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT

#####ILLPCR2-ABMod(G)

5' C-A-A GCA GAA GAC GGC ATA CGA GAT GTG ACT GGA GTT CAG ACG TGT GCT CTT CCG ATC TGT A*A*G

- This is a selective Illumina primer, which will reduce the number of fragments sequenced by ~4x. The last base is selective (arbitrarily G - i.e. will sequence fragments with -G i.s.o. -C/T/A). 
- Dashes(-) represent phosphothiolation. This prevents exonucleases from acting on the ends of the fragments, thus pretecting the product prior to sequencing. 

**VERY IMPORTANT**:  Due to different chemistries for the single-end and paired-end runs, three nucleotides are added to Alan's primers which are essential for the PE kit to work. (MiSeq runs only PE kits). 




####0. Preparation of specialised reagents:

*Adapter P1*: Barcoded *Eco*RI primer combinations. 

1. Combine 1ul P1.1 + 1ul P1.2 (100uM stock) with 98ul water to make 100ul of 1uM annealed, double stranded adapter stock. 
2. Heat to 95°C for 5 min and slowly cool to room temperature (no faster than 2°C/min). 

- Keep the adapters organised in plate format for easy use in future reactions. 
- Adapters can be kept at 4°C when in use. (Recommendations suggest no longer than 2 weeks). For long term storage keep at -80°C. 

*Adapter P2*: *MSe*I-bar adapters (for use with high-density genotyping)

1. Combine 100ul of P2.1 + P2.2. (100uM stock) with 800ul water to make 1000ul of 10uM stock.
2. Heat to 95°C for 5min and slowly cool to room temperature to anneal the oligos into double-stranded adapter (no faster than 2°C/min). 

Storage same as above. 

It is recommended to make fresh adapters for every library prep. For a 48 sample library (+20%) make up 100ul of P2

*PCR primers*

1. Mix 50ul of each primer (100uM stock) with 900ul water to make a working solution containing 5uM of each primer. 

Storage same as above. 

Prepare 10ul per primer per library. i.e. 80ul per flowcell.

####1. Sample preparation

1. Measure DNA concentration using the Qubit. Use 1ul of DNA: 199ul High Sensitivity (HS) Buffer + Dye, according to the manufacturers instructions. Vortex before measuring. 
2. Start with 12ug of DNA per library. For 48 samples this equates to 250ng of DNA/sample, made up to 88ul with water. 

- Samples with low concentrations can be evaporated using the SpeedVac. (~-15ul /25min at RT). 
- Re-measure DNA concentrations with Qubit after speedvac and evaporate for longer if needed. 
- Be careful of overdrying!



####2. Restriction Digestion

- For high density libraries, *MSe*I and *Eco*RI can be used. This results in ~20MB resolution in Rana temporaria. 
- For lower density libraries, Alan Brelsford uses *Sbf*I/*MSe*I. (Adapters will have to be ordered)
- GDC has only *Eco*RI adapters for ddRAD


1. Prepare Master Mix I

- 16ul prepared per sample
- Use low binding eppendorf tubes due to high viscosity of the enzymes. 
- Use the Socorex pipette (10-200ul) which increases accuracy of pipetting.

*MasterMix1*

Reagent	|1x	|1x (ul)	|55x (ul)|
:-----|:---:|:---:|:---:|
Cutsmart buffer (10x) |1x  |1.6		|88
*Mse*I (10000U/ml)    |40U   |4.0	|220
*Eco*RI (20000U/ml)	|40U|	2.0	|110
Water (UHQ) | | 8.4 | 462

2. Mix by vortexing and centrifuge briefly.
3. Add 16ul of MM1 to each well 

  - Use the yellow non-filter tips and the manual multichannel. 
  - This gives a total of 100ul

4. Seal plate. Vortex and spin down. 
5. Digest at 37°C for 3hrs and cool to RT 

  - Do not heat kill as this skews base composition.
  - 20U of enzyme digests 1mg DNA in 30min, but GeCKo have found that samples don't always digest completely 
  - There is uneven digestion between samples
  
6. Before moving to the next step, cool restriction digest to room temperature. Alternatively, product can be stored at 4°C overnight. 



####3. Gel (optional)
1. Prepare 1% Agarose gel. 
2. Run 2ul of RD mix + 2ul loading dye for 30min at 100V. 
3. Gel should show a smear with no distinct bands. Make a note of where the highest concentration of DNA is. For R. temp the brightest region is between 200-700bp, with the size to select ~450-500bp. 

- Single digest of samples: *Eco*RI digested samples look very similar to the whole genomic DNA. *Eco*RI can be tested by digesting a vector. 



####4. Qiagen MinElute PCR cleanup: Restriction digest

The columns clean the Restriction Digest, but doesn't get rid of the adapter dimer. This needs to be cleaned with AMPure cleanup at the end of the library prep. The best is to optimise for every species. 

1. Add 300ul of Buffer 1 to the restriction digest in a deep well plate.
2. Follow the manufacturer's instructions
3. Elute twice in 18ul buffer for a total elute of 36ul. 



####5. Normalise samples

1. Determine the DNA concentration of each cleaned digest using Qubit dsDNA BR (1ul)
2. Normalise the samples (100-300ng/sample is optimal)

*NB* This step is essential for ensuring even representation of samples in the sequencing. 



####6. Adapter ligation

1. Defrost *Eco*RI adapters (in plate form) and *MSe*I-bar adapters (in one eppie in the -80°C freezer) in the fridge or on ice.
2. Make MasterMix II. 

Reagent	|1X	|1X (ul)	|55X (ul)
:---|:--:|:--:|:--:|
Cutsmart 10x	|1x	|4.5	|247.5
rATP 100mM	|	0.5	|27.5
MSeI Adapter (10uM)		|1.6	|88
T4 Ligase (2000U/ml)	|	0.2|	11.0
Water + DNA	|||	36.6	
		*Total*|||6.8ul/sample	
 
3. Add 6.8ul of MMII to each well containing 36.6ul DNA + water. Add 1.6ul of the EcoRI (1uM working soln) adapters to each corresponding sample well for a total volume of 45ul. 
4. Ligate in PCR cycler:
1 hr. @ 23°C.
 10 min @ 65°C
Cool to 4°C at a rate of 2°C/90sec. 

***This can be run overnight. Most protocols opt for 30min at 23°C


####7. Pool and MinElute cleanup
1. Pool sets of 6 ligated samples together, for a total volume of 240ul
2. Add 1200ul of buffer 1 in a deep well plate and mix with pooled samples. 
3. Apply 720ul to the MinElute column. Repeat with the rest of the mixture. 
4. Follow manufacturer's protocol
5. Elute in 11ul buffer (spin twice)
6. Combine all the purified pools together (i.e. 8 pools per library) and purify again with MinElute column. 
•	Per library: DNA (88ul) + buffer 1 (440ul)
7. Elute in 20ul buffer
8. Measure DNA concentration with the Qubit BR. (1ul DNA). 1ul can be kept to run on the bioanalyzer if needed. 

>> expect 550-650ng/ul  
This is very high, but is inflated due to the adapter dimer.

####8. Size selection (caliper)
1.	Prepare DNA to run on the caliper: 3ul DNA + 12ul water + 3ul Sample buffer = 18ul.
(1ug max can be loaded, but it seems like the DNA yield is not more at these volumes).
Prepare 2 x per library - the yield from 2 lanes per library is needed for sufficient library. 
 **Store the remaining library at -20°C for future use
2.	Size select 340bp +/- 15% on the caliper run. This should yield size fragments ~580bp (Migration deviates from expected due to excessive adapter dimer in the mix)
3.	Add 21ul of collection buffer, so that 20ul can be collected after run
4.	Prepare 2 lanes (18ul, ~1500ng each) per library to get enough yield at the end. 
5.	Eluate should be collected within 5min of end of the run. 
Migration on the caliper is faster due to forked primers. After PCR fragments are double stranded and migrate at the correct speed. 
Remember than 10% of 500bp is 50bp, so if selecting 300bp, choose 15% range...
Caliper		340 +/- 15% = avg. fragment size of 450bp
		400 +/- 10% = avg. fragment size of 550bp(?)
		480 +/- 10% = avg. fragment size of 650bp



####9. PCR 
1. Collect 20ul of elute from the caliper. Add the two lanes together for a total volume of 40ul. If only one lane is used, make up the volume to 40ul using water.
2. Make Master Mix III
Reagent	1X (ul)
Template DNA	40
Illumina Primer1 (5uM)	8
ABMod Primer (Ill2) (5uM)	8
Phusion MM	88
Water	32
	176ul

3. Split into 12 reactions of 13-14ul each. 
4. Run 14 PCR cycles: 98°C 30s/ 98°C 10s, 65°C 30s, 72°C 30s. 
5. Leave AMPure beads on bench to warm to RT while PCR is running. 

####10. Final AMPure cleanup
**AMPure is needed to get rid of the large adapter dimer peak
**For my libraries a cleanup of 1:1 followed by 0.8:1 is optimal. But this can be optimised for new species. 
**Sample volumes should be measured before starting: since selection is based on bead to library ratios, clean-ups are very sensitive to variations in sample volume
**If final libraries deviate from the optimal size, clean again with 0.7:1 AMPure selection. Be sure to measure the starting library volumes to use the appropriate volume of beads per sample. Elute in 20ul water. 
AMPure protocol below:
1. Combine all PCR reactions.
2. Use a 1X AMPure mix (i.e. 176ul library : 176ul beads) ***measure library volumes!
3. Elute in 40ul water
4. Follow with a 0.8 X bead clean up (40ul library : 32ul beads) **measure library volumes!
5. Elute in 20ul water

#####AMPure Protocol: 
The smaller the ratio, the wider the range of fragments removed. 1:1 ratio removes <200bp, while 0.8:1.0 (beads:DNA) will remove larger fragments. 
1. Leave AMPure beads on the bench for 30min to equilibrate to RT. 
2. Make fresh 80% EtOH. (For 48 samples you'll need 20ml. i.e. 14ml Abs EtOH + 6ml H2O)
3. Add 1 volumes (176ul) of AMPure bead mix to each well. Mix by pipetting up and down 10 times. 
4. Incubate on the bench for 5min at RT.
5. Place on magnetic stand for 5 min.
6. Carefully remove the supernatent and discard. 
•	Remove liquid and discard. 
•	Leave some liquid in the bottom rather than removing some beads
5. Wash the beads 2x with 80% EtOH while still on the magnetic plate: 
Add enough 80% EtOH to each well to cover the bead smear (~200ul). Incubate at RT  for 1min. Remove last EtOH with long tips. 
6. Still on the magnetic plate, leave the plate to dry at RT for 5 - 10min. 
7. Elute in 20ul water. 
•	Mix by pipetting ~10 times each. 
•	Incubate on bench for 5min. 
8. Place on the magnetic plate for 2min. Remove 20ul DNA to a LoBind eppie. 

####11. Library QC
1. Measure DNA concentration using the Qubit (HS). 
2. Assess peak using the BioAnalyzer (HS DNA chip). The final DNA concentration should be at least 1ng/ul.
3. qPCR to determine the amount of ligated adapter

*** calculation for molarity from Qubit measure: 
xng/ul x 10^6ul/1L x 1nmol/660ng x 1/N = nM  
(where N = avg size in bp)  
***Bioanalyzer: check the shape of the curve (should be steep) & that there are no small peaks (i.e. adapter dimer). Molarity and size can be measured by the Bioanalyzer.
FGCZ asks for at least 15ul of 2nM
