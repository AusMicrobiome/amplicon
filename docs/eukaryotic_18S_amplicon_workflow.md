# Eukaryotic 18S rRNA gene Amplicon Analysis Workflow

This document outlines the workflow required to analyse 18S rRNA gene amplicon sequences to produce Amplicon Sequence Variant (ZOTU) information for the Australian microbiome database.

This workflow covers both 18S variable region 4 (18Sv4) amplified by the 18S_V4f/18S_V4r primer set and 18S variable region 9 (18Sv9) amplified by the ILM_Euk_1391f/ILM_EukBr primer set.

The ZOTU analysis is performed using a combination of per sample and per sequencing run (sequencing plate) basis. The workflow allows for the inclusion of known "prior" sequences that can be added following per sample and per plate denoising steps. 

***Note: In this workflow, the use of 18Svn represents the 18S region being analysed (e.g., 18Sv4 or 18Sv9)***

The workflow consists of thefollowing stages:

 1. **Sequence preparation**
    - Trim paired end reads (18Sv4 only)
    - Merge paired end reads (non-merged reads are discarded)
    - Convert fastq file format to fasta file format and rename files
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
    - Classify and remove sequences in the wrong orientation
    - Replace arbitrary ZOTU ID's with the sequence itself in the table index
5. **Prepare the single dataset**
    - Merge tables into a single table
    - Remove controls from the abundance tables, to create separate sample and control datasets
    - Make a fasta file from unique ZOTUs in the abundance table
    - Classify Sequences

**Software used**

The following software is used in the workflow:

1. Seqtk (https://github.com/lh3/seqtk)
2. FLASH2 (Magoc and Salzberg, 2011)
3. FASTX (http://hannonlab.cshl.edu/fastx_toolkit/)
4. USEARCH (Edgar 2010)
5. Mothur (Schloss, et al., 2009)
6. QIIME2 (Bolyen et al., 2019)
7. Python3

### Sequence preparation

**Merge the paired end reads**

For 18Sv4 amplicons, primer removal is performed prior to merging using **seqTk** by hard trimming 20 nucleotides from the 5′ end of R1 sequences and 21 nucleotides from the 5′ end of its respective R2 paired end read.

Sequences are merged using **FLASH2** (Magoc and Salzberg, 2011). 

18Sv4 paired end reads are merged using **FLASH2** with the following arguments:

    --min-overlap=50 --max-overlap=160 --allow-outies

18Sv9 paired end reads are merged using **FLASH2** with the following arguments:

    --min-overlap=50 --max-overlap=120 --allow-outies

Following merging, the merge quality is manually checked by examining the FLASH log file for the percentage of reads that were merged. Plates with low merge rates (< 70%) are manually checked to see if the alignments can be improved. Unmerged reads are discarded.

**Fastq files are converted to fasta format**

**seqTk** is used to convert merged reads from fastq to fasta format:

    seqtk seq -A sampleID_*.fastq > sampleID_*.fasta

**File naming**

For sequence files with AM sampleIDs, file names are changed to the following format:

`sampleID_plateID.fasta`

For sequence files downloaded from NCBI, file names are changed to a similar format using the NCBI BioSampleID and BioProjectID, as below.

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

An abundance table of all unique sequences in each sample on the plate is prepared. Unique sequences are identified from the merged R1/R2 sequences using **FASTX** with the following **USEARCH** command:

    usearch -fastx_uniques sampleID_plateID.fasta -fastaout sampleID_plateID_uniques.fasta -sizeout

Unique sequences are then converted into a 3 column tab separated abundance table containing the columns:

`seq\tSample\tAbundance`

*Unique sequences per sample (non-Denoised or quality filtered) are provided as a data output and are available by request through the Australian Microbiome Website.*

### 3. Sample-wise denoising

Each sample file from the sequencing run (e.g., sampleID_plateID.fasta) is denoised individually using the following steps.

**Quality screening, ZOTU calling and sequence mapping**

The first step performs some quality control on the sequences, with each amplicon region having different parameters.

For 18Sv4, the quality screen removes sequences that are too short, have ambiguous bases, or have more than 12 homopolymers or do not meet a minimum length:

    mothur "#set.dir(modifynames=F); screen.seqs(fasta= sampleID_plateID.fasta, minlength=300, maxambig=0, maxhomop=12, processors=10)"

For 18Sv9, the quality screen removes sequences that have ambiguous bases, or have more than 12 homopolymers:

    mothur "#set.dir(modifynames=F); screen.seqs(fasta= sampleID_plateID.fasta, maxambig=0, maxhomop=12, processors=10)"

Following quality screening, the remainder of the ZOTU workflow follows the same parameters for both amplicon types

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

**Concatenate all sequences per sequencing run into a single file**

All sample fasta files for each plate are concatenated into a single file ZOTU calling. The resulting file name is standardised to the format:

`plateID_all_18Svn.fasta`

**Quality screening, ZOTU calling and sequence mapping**

The first step performs some quality control on the sequences, with each amplicon region having different parameters:

For 18Sv4, quality screening removes sequences that are too short, have ambiguous bases, or have more than 12 homopolymers:

    mothur "#set.dir(modifynames=F);summary.seqs(fasta=plateID_all_18Sv4.fasta, processors=10); screen.seqs(fasta=current, minlength=300, maxambig=0, maxhomop=12, processors=10); summary.seqs()"

For 18Sv9 quality screening removes sequences have ambiguous bases, or have more than 12 homopolymers:

    mothur "#set.dir(modifynames=F);summary.seqs(fasta=plateID_all_18Sv9.fasta, processors=10); screen.seqs(fasta=current, maxambig=0, maxhomop=12, processors=10); summary.seqs()"

Following quality screening, the remainder of the ZOTU workflow follows the same parameters for both amplicon types.

**Next reads are dereplicated:**

    usearch -fastx_uniques plateID_all_18Svn.good.fasta -fastaout plateID_all_18SVn.good_uniques.fasta -sizeout

**Sort unique reads by abundance:**

    usearch -sortbysize plateID_all_18Svn.good_uniques.fasta -fastaout plateID_all_16S.good_sorted_uniques.fasta -sizeout

**ZOTUs are called by *UNOISE3*, from sequences that have => 8 representatives:**

    usearch -unoise3 plateID_all_18Svn.good_sorted_uniques.fasta -ZOTUs plateID_all_18Svn.good_sorted_uniques_ZOTUs.fasta -ampout plateID_all_18Svn.good_sorted_uniques_ampout.fasta -tabbedout plateID_all_18Svn.good_sorted_uniques_unoise3.txt -minsize 8

Combine the ZOTUs obtained from sample-wise and plate-wise analysis with a defined set of "prior" sequences into a single fasta file. The resulting file name is standardised to the format:

`plateID_all_18Svn.good_sorted_uniques_ZOTUs_renamed_SWpriors.fasta`

**Dereplicate the combined ZOTU file:**

    usearch -fastx_uniques plateID_all_18Svn.good_sorted_uniques_ZOTUs_renamed_SWpriors.fasta -fastaout plateID_all_18Svn.good_sorted_uniques_ZOTUs_renamed_SWpriors_uniq.fasta

**Map reads to ZOTUs to generate abundances**

Reads are mapped against the ZOTUs using **USEARCH**. Note the termination conditions on the mapping run (**-maxaccepts 0**), this seems to be required to ensure the best match is found and to produce consistent results when adding multiple plates together, as we do later.

    usearch -otutab plateID_all_18Svn.fasta -ZOTUs plateID_all_18Svn.good_sorted_uniques_ZOTUs_renamed_SWpriors_uniq.fasta -otutabout plateID_all_18Svn.good_sorted_uniques_PWSW_ZOTUtab_MA0.txt -mapout plateID_all_18Svn.good_sorted_uniques_PWSW_zmap_MA0.txt -maxaccepts 0 -threads 10

**Replace ZOTU table indexes with ZOTU sequence**

Currently the tables have an arbitrary OTU number as the index, replace this with the sequence that the arbitrary number represents. These sequences are unique strings and allow tables to be merged etc. downstream easily. They also negate the need to maintain a dictionary of ZOTUs and the sequences they represent.

**Classify and remove flipped sequences**

A final QC step is performed to remove likely erroneous sequences. The ZOTUs are classified, with those that do not align to the 18S Silva132 database (Quast et al., 2013; Yilmaz et al 2014; Glöckner 2017) in the correct orientation being removed. Those that need to be "flipped" to a new orientation are likely errors, since we know the reads should be in correct orientation for their respective primer sets `18S_V4f/18S_V4`r or `ILM_Euk_1391f/ILM_EukBr`. This step typically removes < 10 ZOTUs from the database.

**A. Classify the seqs against silva132 database:**

    mothur "#set.dir(modifynames=F); classify.seqs(fasta=plateID_all_18Svn.good_sorted_uniques_PWSW_zotutab_relabelled_MA0.fasta, reference=silva.nr_v132.align, taxonomy=silva.nr_v132.tax, cutoff=60, probs=FALSE)"

**B. Use the \*acnos.flip list to remove sequences from the table**

The above produces the final abundance table, with sequences as index for each sequencing run. These tables are then combined as below to produce a single dataset.

### 5. Prepare the single dataset

Now we have a ZOTU abundance table for each plate, with ZOTU's as row and sampleID_plateID as column headers. To prepare this data for ingest into the AM database the following steps are carried out:

1. Each table is converted from short to long format (from rectangular to 3 column, with columns ZOTU, sampleID, Abundance)
2.  All of these 3 column tables are concatenated into a single table
3. Controls and samples are split into separate tables
4. Sequencing run ID's (plateIDs) are removed from the column headers and any sample sequenced more than once has the OTUs grouped and abundances summed to give a single abundance per sample
5. A fasta file of unique ZOTU's is created from all ZOTU's in this final table
6. Sequences are classified

**Classify the sequences**

Final taxonomic tables are produced by classifying sequences using a
variety of taxonomic databases and classification methodologies.

Databases used for eukaryotic 18S:

-   Silva 138 (Quast et al., 2013; Yilmaz et al., 2014; Glöckner 2017)

-   Silva 132 (Quast et al., 2013; Yilmaz et al., 2014; Glöckner 2017)

-   PR2v`<number>`* (Guillou, et al., 2013; del Campo et al., 2018; Vaulot, et. al., 2021)

**Database version options used for taxonomic assignments are provided in the AM data portal (https://data.bioplatforms.com/bpa/otu/) taxonomy selection pulldown. Information on the version used can also be found in the `info.txt` file contained in AM data portal download packets.*

Classifiers are used with the following arguments:

**QIIME2** SKlearn Bayesian classifier:

    --p-n-jobs 10 --p-confidence 0.6 --p-read-orientation same --verbose

**MOTHUR** wang Bayesian classifier:

    cutoff=60, probs=FALSE, processors=8

**USEARCH** usearch_global nearest neighbour classifier:

    -query_cov 1.0 -id 0.8 -maxaccepts 100 -maxrejects 100 -maxhits 1 -strand plus -threads 8

**References**
1. Seqtk available at: <https://github.com/lh3/seqtk> (last accessed 23 Jan 2019).
2. Magoc, T. and Salzberg, S. (2011). FLASH: Fast length adjustment of short reads to improve genome assemblies. Bioinformatics 27(21): 2957-2963.
3. FASTX available at: (<http://hannonlab.cshl.edu/fastx_toolkit/>) (last accessed 23 Jan 2019).
4. Edgar, R.C., (2010), Search and clustering orders of magnitude faster than BLAST, Bioinformatics 26(19): 2460-2461.
5. Schloss, P.D., et al., (2009). Introducing mothur: Open-source, platform-independent, community-supported software for describing and comparing microbial communities. Appl Environ Microbiol 75(23):7537-7541.
6. Quast C, Pruesse E, Yilmaz P, Gerken J, Schweer T, Yarza P, Peplies J, Glöckner FO (2013) The SILVA ribosomal RNA gene database project: improved data processing and web-based tools. Nucl. Acids Res. 41 (D1): D590-D596.
7. Yilmaz P, Parfrey LW, Yarza P, Gerken J, Pruesse E, Quast C, Schweer T, Peplies J, Ludwig W, Glöckner FO (2014) The SILVA and "All-species Living Tree Project (LTP)" taxonomic frameworks. Nucl. Acids Res. 42:D643-D648.
8. Glöckner FO, Yilmaz P, Quast C, Gerken J, Beccati A, Ciuprina A, Bruns G, Yarza P, Peplies J, Westram R, Ludwig W (2017) 25 years of serving the community with ribosomal RNA gene reference databases and tools. J. Biotechnol. 10:169-176.doi: 10.1016/j.jbiotec.2017.06.1198
9. Guillou, L., Bachar, D., Audic, S., Bass, D., Berney, C., Bittner, L., Boutte, C. et al., (2013). The Protist Ribosomal Reference database (PR^2^): a catalog of unicellular eukaryote Small Sub-Unit rRNA sequences with curated taxonomy. Nucleic Acids Res. 41:D597--604.
10. del Campo J., Kolisko M., Boscaro V., Santoferrara LF., Nenarokov S., Massana R., Guillou L., Simpson A., Berney C., de Vargas C., Brown MW., Keeling PJ., Wegener Parfrey L. (2018).EukRef: Phylogenetic curation of ribosomal RNA to enhance understanding of eukaryotic diversity and distribution. PLOS Biology 16:e2005849. DOI: 10.1371/journal.pbio.2005849.
11. Vaulot, D., Mahé, F., Bass, D., & Geisen, S. (2021).pr2-primer : An 18S rRNA primer database for protists. Molecular Ecology Resources, in press. DOI: 10.1111/1755-0998.13465.
12. Bolyen E, Rideout JR, Dillon MR, Bokulich NA, Abnet CC, Al-Ghalith GA, Alexander H, Alm EJ, Arumugam M, Asnicar F, Bai Y, Bisanz JE, Bittinger K, Brejnrod A, Brislawn CJ, Brown CT, Callahan BJ, Caraballo-Rodríguez AM, Chase J, Cope EK, Da Silva R, Diener C, Dorrestein PC, Douglas GM, Durall DM, Duvallet C, Edwardson CF, Ernst M, Estaki M, Fouquier J, Gauglitz JM, Gibbons SM, Gibson DL, Gonzalez A, Gorlick K, Guo J, Hillmann B, Holmes S, Holste H, Huttenhower C, Huttley GA, Janssen S, Jarmusch AK, Jiang L, Kaehler BD, Kang KB, Keefe CR, Keim P, Kelley ST, Knights D, Koester I, Kosciolek T, Kreps J, Langille MGI, Lee J, Ley R, Liu YX, Loftfield E, Lozupone C, Maher M, Marotz C, Martin BD, McDonald D, McIver LJ, Melnik AV, Metcalf JL, Morgan SC, Morton JT, Naimey AT, Navas-Molina JA, Nothias LF, Orchanian SB, Pearson T, Peoples SL, Petras D, Preuss ML, Pruesse E, Rasmussen LB, Rivers A, Robeson MS, Rosenthal P, Segata N, Shaffer M, Shiffer A, Sinha R, Song SJ, Spear JR, Swafford AD, Thompson LR, Torres PJ, Trinh P, Tripathi A, Turnbaugh PJ, Ul-Hasan S, van der Hooft JJJ, Vargas F, Vázquez-Baeza Y, Vogtmann E, von Hippel M, Walters W, Wan Y, Wang M, Warren J, Weber KC, Williamson CHD, Willis AD, Xu ZZ, Zaneveld JR, Zhang Y, Zhu Q, Knight R, and Caporaso JG. (2019) Reproducible, interactive, scalable and extensible microbiome data science using QIIME 2. Nature Biotechnology 37: 852--857.<https://doi.org/10.1038/s41587-019-0209-9>.
