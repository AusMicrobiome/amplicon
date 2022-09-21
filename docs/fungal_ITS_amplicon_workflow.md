# Fungal ITS Amplicon Analysis Workflow

This document outlines the workflow required to analyse amplicon sequences of the internal transcribed spacer (ITS) region located between the small and large rRNA subunits to produce Amplicon Sequence Variant (ZOTU) information for the Australian microbiome database.

This workflow covers both ITS1 and ITS2 regions from amplicons derived from primers targeting the fungal ITS1 and ITS4 regions (ITS1F and ITS4).

The ZOTU analysis is performed both on a per sample and per sequencing run (sequencing plate) basis. The workflow allows for the inclusion of known "prior" sequences that can be added following per sample and per plate denoising steps. 

***Note: In this workflow, the use of  ITSn represents the ITS region being analysed (e.g., ITS1 or ITS2)***

The workflow consists of the following stages:

1. **Sequence preparation**
   - Convert R1 and R2 fastq files to fasta file format
   - Generate the reverse complement of R2 reads (ITS2 only)
   - Identify and isolate putative fungal ITS1 and ITS2 regions from R1 and R2 reads and rename files
   - Add sampleID, runID and "sample=" information to the sequence headers
2. **Generate unique sequence dataset**
   - Generate unique sequences
   - Convert unique sequences to 3 column abundance table
3. **Sample-wise denoising**
   - Quality screening and ZOTU calling on individual samples on a plate
   - Concatenate all sample-wise ZOTU in the sequencing run into a single file
4. **Plate-wise denoising**
   - Concatenate all sequences per sequencing run into a single file
   - Quality screening and ZOTU calling of concatenated samples on the plate
   - Concatenate plate-wise ZOTUs, sample-wise ZOTUs and prior sequences into a single file
   - Dereplicate duplicated ZOTU sequences and sequence mapping
   - Classify and remove flipped sequences
   - Replace arbitrary ZOTU ID's with the sequence itself in the table index
5. **Prepare the single dataset**
   - Merge tables into a single table
   - Remove controls from the abundance tables, to create separate samples and control datasets
   - Make a fasta file from unique ZOTUs in the abundance table
   - Classify Sequences

**Software used**

The following software is used in the workflow:

1. Seqtk (https://github.com/lh3/seqtk)
2. ITSx (Bengtsson-Palme et al., 2013)
3. FASTX (http://hannonlab.cshl.edu/fastx_toolkit/)
4. USEARCH (Edgar 2010)
5. Mothur (Schloss, et al., 2009)
6. Python3

### 1. Sequence preparation

**Fastq files are converted to fasta format**

**seqTk** is used to convert R1/R2 reads from fastq to fasta format:

    seqtk seq -A sampleID_*.fastq > sampleID_*.fasta

In addition, for ITS2 SeqTk is used to generate the reverse complement of R2 reads:

    seqtk seq -r sampleID_*_R2*.fasta > sampleID_*_R2*_RC.fasta

**Identify and isolate ITS1 and ITS2 regions**

ITSx (Bengtsson-Palme et al., 2013) is then used to identify and isolate fungal ITS1 and ITS2 regions from neighbouring ribosomal genes (SSU, 5S and LSU rRNA sequences). Arguments used for ITSx are as follows:

    -t F --complement F --preserve T --partial 100 --save_regions ITSn --detailed_results T

R1 and R2 reads not identified as ITS by ITSx are discarded

**File naming**

For sequence files with AM sampleIDs, file names are changed to the following format:

`sampleID_plateID.fasta`

For sequence files downloaded from NCBI, file names are changed to a similar format using the NCBI BioSampleID and BioProjectID, as below:

`BioSampleID_BioProjectID.fasta`

The sampleID of the mock communities and negative controls on each plate are standardised to the following naming convention (all amplicon variants listed for completeness):

- 16S Bacteria mock communities: **BACMOCK**
- A16 Archaeal mock communities: **ARCMOCK**
- 18Sv4 Eukaryote mock communities: **EUKMOCKV4**
- 18Sv9 Eukaryote mock communities: **EUKMOCKV9**
- ITS Fungal mock communities: **FUNMOCK**
- Negative control: **NEG**
- STAN: **STAN**
- MSA-1002 (ATCC) 20 Strain even mix genomic material: **ATCC1002MOCK**
- MSA-1010 (ATCC) Mycobiome genomic DNA mix: **ATCC1010MOCK**

**Add sample and file name to the sequence headers**

Sample identifiers are added to the header of each sequence for downstream processing. As each fasta is now named **sampleID_plateID.fasta** we simply add the file name without the extension to the sequence header. At this stage we also add any other information and delimiters that downstream programs will likely require. For **USEARCH** we add "sample=" and ";", for **QIIME** we add "_". A bash script to perform this can be found at https://raw.githubusercontent.com/AusMicrobiome/misc_tools/master/add_sample_name.sh

### 2. Generate unique sequence dataset

An abundance table of all unique sequences in each sample on the plate is prepared. Unique sequences are identified from the output of ITSx using **FASTX** with the following **USEARCH** command:

    usearch -fastx_uniques sampleID_plateID.fasta -fastaout sampleID_plateID_uniques.fasta -sizeout

Unique sequences are then converted into a 3 column tab separated abundance table containing the columns:

`seq\tSample\tAbundance`

*Unique sequences per sample (non-Denoised or quality filtered) are provided as a data output and are available by request through the Australian Microbiome Website.*

### 3. Sample-wise denoising

Each sample file from the sequencing run (e.g., sampleID_plateID.fasta) is denoised individually using the following steps.

**Quality screening, ZOTU calling and sequence mapping**

The first step performs some quality control on the sequences:

    mothur "#set.dir(modifynames=F); screen.seqs(fasta= sampleID_plateID.fasta, maxambig=0, maxhomop=12, processors=10)"

**Reads are dereplicated:**

    usearch -fastx_uniques sampleID_plateID.good.fasta -fastaout sampleID_plateID.good_uniques.fasta -sizeout

**Sort unique reads by abundance:**

    usearch -sortbysize sampleID_plateID.good_uniques.fasta -fastaout sampleID_plateID.good_sorted_uniques.fasta -sizeout

**ZOTUs are called by *UNOISE3*, from sequences that have => 8 representatives:**

    usearch -unoise3 sampleID_plateID.good_sorted_uniques.fasta -ZOTUs sampleID_plateID.good_sorted_uniques_ZOTUs.fasta -minsize 8

**Concatenate all sample-wise ZOTUs into a single file**

All ZOTU files for each sample are concatenated into a single file, this file is later combined with ZOTUs obtained from the per-plate analysis. The resulting file name is standardised to the format:

`plateID_all_SW_ZOTUs.fasta`

### 4. Plate-wise denoising

**Concatenate all sequences into a single file**

All sample fasta files for each plate are concatenated into a single file ZOTU. The resulting file name is standardised to the format:

`plateID_all_ITSn.fasta`

**Quality screening, ZOTU calling and sequence mapping**

The first step removes sequences that have ambiguous bases, or have more than 12 homopolymers:

    mothur "#set.dir(modifynames=F);summary.seqs(fasta=plateID_all_ITSn.fasta, processors=10); screen.seqs(fasta=current, maxambig=0, maxhomop=12, processors=10); summary.seqs()"

**Next reads are dereplicated:**

    usearch -fastx_uniques plateID_all_ITSn.good.fasta -fastaout plateID_all_ITSn.good_uniques.fasta -sizeout

**Sort unique reads by abundance:**

    usearch -sortbysize plateID_all_ITSn.good_uniques.fasta -fastaout plateID_all_ITSn.good_sorted_uniques.fasta -sizeout

**ZOTUs are called by *UNOISE3*, from sequences that have => 8 representatives:**

    usearch -unoise3 plateID_all_ITSn.good_sorted_uniques.fasta -zotus plateID_all_ITSn.good_sorted_uniques_zotus.fasta -ampout plateID_all_ITSn.good_sorted_uniques_ampout.fasta -tabbedout plateID_all_ITSn.good_sorted_uniques_unoise3.txt -minsize 8

Combine the ZOTUs obtained from sample-wise and plate-wise analysis with a defined set of "prior" sequences into a single fasta file. The resulting file name is standardised to the format:

`plateID_all_ITSn.good_sorted_uniques_ZOTUs_renamed_SWpriors.fasta`

**Dereplicate the combined ZOTU file:**

    usearch -fastx_uniques plateID_all_ITSn.good_sorted_uniques_ZOTUs_renamed_SWpriors.fasta -fastaout plateID_all_ITSn.good_sorted_uniques_ZOTUs_renamed_SWpriors_uniq.fasta

**Map reads to ZOTUs to generate abundances:**

Reads are mapped against the ZOTUs using **USEARCH**. Note the termination conditions on the mapping run (**-maxaccepts 0**), this seems to be required to ensure the best match is found and to produce consistent results when adding multiple plates together, as we do later.

    usearch -otutab plateID_all_ITSn.fasta -ZOTUs plateID_all_ITSn.good_sorted_uniques_ZOTUs_renamed_SWpriors_uniq.fasta -otutabout plateID_all_ITSn.good_sorted_uniques_PWSW_ZOTUtab_MA0.txt -mapout plateID_all_ITSn.good_sorted_uniques_PWSW_zmap_MA0.txt -maxaccepts 0 -threads 10

**Replace ZOTU table indexes with ZOTU sequence**

Currently the ZOTU tables have an arbitrary ZOTU number as the index, replace this with the sequence that the arbitrary number represents. These sequences are unique strings and allow tables to be merged etc. downstream easily. They also negate the need to maintain a dictionary of ZOTUs and the sequences they represent.

**Classify and remove flipped sequences**

A final QC step is performed to remove likely erroneous sequences. The ZOTUs are classified, with those that do not align to the UNITEv8 SH (Picard et al., 2018) ITS database in the correct orientation being removed. As we know that the R1 sequences are correctly orientated and the reverse complement of the R2 also puts it into the correct orientation, sequences that need to be "flipped" to a new orientation to obtain the best alignment to the database are most likely errors. This step typically removes < 10 ZOTUs from the database.

**A. Classify the seqs against UNITE ITS database:**

    mothur "#set.dir(modifynames=F); classify.seqs(fasta=${plateID}_all_ITSn.good_sorted_uniques_PWSW_zotutab_relabelled_MA0.fasta, reference=UNITEv8_sh_dynamic.fasta, taxonomy=UNITEv8_sh_dynamic.tax, cutoff=60, probs=FALSE)"

**B. Use the \*acnos.flip list to remove flipped sequences from the table**

The above produces the final abundance table, with sequences as index for each sequencing run. These tables are then combined as below to produce a single dataset.

### 5. Prepare the single dataset

Now we have a ZOTU abundance table for each plate, with ZOTU's as row and sampleID_plateID as column headers. To prepare this data for ingest into the AM database the following steps are carried out:

1. Each table is converted from short to long format (from rectangular to 3 column, with columns ZOTU, sampleID, Abundance)
2. All of these 3 column tables are concatenated into a single table
3. Controls and samples are split into separate tables
4. Sequencing run ID's (plateIDs) are removed from the column headers and any sample sequenced more than once has the OTUs grouped and abundances summed to give a single abundance per sample
5. A fasta file is created from all ZOTU's in this final table
6. Sequences are classified

**Classify the sequences**

Final taxonomic tables are produced by classifying sequences using the below taxonomic database(s) and a variety classification methodologies.

Databases used for eukaryotic fungal ITS:

-   UNITEv8 SH (Picard et al., 2018)

Classifiers are used with the following arguments:

**MOTHUR** wang Bayesian classifier:

    cutoff=60, probs=FALSE, processors=8

**USEARCH** usearch_global nearest neighbour classifier:

    -query_cov 1.0 -id 0.8 -maxaccepts 100 -maxrejects 100 -maxhits 1 -strand plus -threads 8

**References**
1. Seqtk available at: <https://github.com/lh3/seqtk> (last accessed 23 Jan 2019).
2. Bengtsson-Palme, J., Veldre V., Ryberg M., Hartmann M., Branco S., Wang Z., Godhe A., Bertrand Y., De Wit P., Sanchez M., Ebersberger I., Sanli K., de Souza F., Kristiansson E., Abarenkov K., Eriksson K.M, Nilsson R.H.(2013). ITSx: Improved software detection and extraction of ITS1 and ITS2 from ribosomal ITS sequences of fungi and other eukaryotes for use in environmental sequencing. Methods in Ecology and Evolution, 4: 914-919.
3. FASTX available at: (<http://hannonlab.cshl.edu/fastx_toolkit/>) (last accessed 23 Jan 2019).
4. Edgar, R.C., (2010), Search and clustering orders of magnitude faster than BLAST, Bioinformatics 26(19): 2460-2461.
5. Schloss, P.D., et al., (2009). Introducing mothur: Open-source, platform-independent, community-supported software for describing and comparing microbial communities. Appl Environ Microbiol 75(23):7537-7541.
6. Nilsson RH, Larsson K-H, Taylor AFS, Bengtsson-Palme J, Jeppesen TS, Schigel D, Kennedy P, Picard K, Glöckner FO, Tedersoo L, Saar I, Kõljalg U, Abarenkov K. (2018). The UNITE database for molecular identification of fungi: handling dark taxa and parallel taxonomic classifications. Nucleic Acids Research, DOI: 10.1093/nar/gky1022.
