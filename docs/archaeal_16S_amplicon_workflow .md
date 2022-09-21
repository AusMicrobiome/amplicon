# Archaeal 16S rRNA gene Amplicon Analysis Workflow

This document outlines the workflow required to analyse 16S rRNA amplicon sequences for Archaea (A2f - 519r) to produce Amplicon Sequence Variant (ZOTU) information for the Australian Microbiome database.

The ZOTU analysis is performed using a combination of per sample and per sequencing run (sequencing plate) basis. The workflow allows for the inclusion of known "prior" sequences that can be added following per sample and per plate denoising steps. 

The workflow consists of the following stages:

1. **Sequence preparation**
   - Merge paired end reads (non-merged reads are discarded)
   - Convert fastq file format to fasta file format and rename files
   - Add sampleID, runID and "sample=" information to the sequence headers
   - Trim forward and reverse primers from merged reads
2. **Generate unique sequence dataset**
   - Generate unique sequences
   - Convert unique sequences to 3 column abundance table**
3. **Sample-wise denoising**
   - Quality screening and ZOTU calling on individual samples on a plate
   -  Concatenate all sample-wise ZOTU in the sequencing run into a single file
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

1. FLASH2 (Magoc and Salzberg, 2011)
2. SeqTk (https://github.com/lh3/seqtk)
3. Cutadapt (Martin, [DOI:10.14806/ej.17.1.200 (http://dx.doi.org/10.14806/ej.17.1.200))
4. FASTX (http://hannonlab.cshl.edu/fastx_toolkit/)
5. USEARCH (Edgar, 2010)
6. Mothur (Schloss, et al., 2009)
7. QIIME2 (Bolyen et al., 2019)
8. Python3

### 1, Sequence preparation

**Merge the paired end reads**

Paired end reads are merged using **FLASH2** (Magoc and Salzberg, 2011). **FLASH2** is run with the following arguments:

    --min-overlap=30 --max-overlap=250

Following merging, the merge quality is manually checked by examining the **FLASH2** log file for the percentage of reads that were merged. Plates with low merge rates (< 70%) are manually checked to see if the alignments can be improved. Unmerged reads are discarded.

**Fastq files are converted to fasta format**

**seqTk** is used to convert merged reads from fastq to fasta format:

    seqtk seq -A sampleID_*.fastq > sampleID_*.fasta

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

**Remove forward and reverse primers from the merged reads**

Primers are removed from the merged reads using **Cutadapt** v2.9 with the following arguments:

**Reverse primer removal:**

    --cores=0 -a AGCMGCCGCGGTAATWCX -O 17 -e 0.01

**Forward primer removal:**

    --minimum-length 1 --cores=0 -g XTTCCGGTTGATCCYGCCGGA -O 20 -e 0.01

### 2. Generate unique sequence dataset

An abundance table of all unique sequences in each sample on the plate is prepared. Unique sequences are identified from the merged R1/R2 sequences using **FASTX** with the following **USEARCH** command:

    usearch -fastx_uniques sampleID_plateID.fasta -fastaout sampleID_plateID_uniques.fasta -sizeout

Unique sequences are then converted into a 3 column tab separated abundance table containing the columns:

`seq\tSample\tAbundance`

*Unique sequences per sample (non-Denoised or quality filtered) are provided as a data output and are available by request through the Australian Microbiome Website.*

### 3. Sample-wise denoising

Each sample file from the sequencing run (e.g., sampleID_plateID.fasta) is denoised individually using the following steps.

**Quality screening, ZOTU calling and sequence mapping**

The first step removes sequences that have ambiguous bases or have more than 12 homopolymers.

    mothur "#set.dir(modifynames=F); screen.seqs(fasta=sampleID_plateID.trimmed.fasta, minlength=380, maxlength=520, maxambig=0, maxhomop=12, processors=10)"

**Reads are dereplicated:**

    usearch -fastx_uniques sampleID_plateID.good.fasta -fastaout sampleID_plateID.trimmed.good_uniques.fasta -sizeout

**Sort unique reads by abundance:**

    usearch -sortbysize sampleID_plateID.trimmed.good_uniques.fasta -fastaout sampleID_plateID.trimmed.good_sorted_uniques.fasta

**ZOTUs are called by *UNOISE3*, from sequences that have => 8 representatives:**

    usearch -unoise3 sampleID_plateID.trimmed.good_sorted_uniques.fasta -ZOTUs sampleID_plateID.trimmed.good_sorted_uniques_ZOTUs.fasta -minsize 8

**Concatenate all sample-wise ZOTUs into a single file**

All ZOTU files for each sample are concatenated into a single file, this file is later combined with ZOTUs obtained from the per-plate analysis. The resulting file name is standardised to the format:

`plateID_all_SW_ZOTUs.fasta`

### 4. Plate-wise denoising

**Concatenate all sequences per sequencing run into a single file**

All sample fasta files for each plate are concatenated into a single file for ZOTU calling. The resulting file name is standardised to the format:

`plateID_all_A16S.fasta`

**Quality screening, ZOTU calling and sequence mapping**

The first step removes sequences that have ambiguous bases or have more than 12 homopolymers or are not within a length range:

    mothur "#set.dir(modifynames=F);summary.seqs(fasta=plateID_all_A16S.trimmed.fasta, processors=10); screen.seqs(fasta=current, minlength=380, maxlength=520, maxambig=0, maxhomop=12, processors=10); summary.seqs()"

**Reads are dereplicated:**

    usearch -fastx_uniques plateID_all_A16S.trimmed.good.fasta -fastaout plateID_all_A16S.trimmed.good_uniques.fasta -sizeout

**Sort unique reads by abundance:**

    usearch -sortbysize plateID_all_A16S.trimmed.good_uniques.fasta -fastaout plateID_all_A16S.trimmed.good_sorted_uniques.fasta -sizeout

**ZOTUs are called by *UNOISE3*, from sequences that have => 8 representatives:**

    usearch -unoise3 plateID_all_A16S.trimmed.good_sorted_uniques.fasta -ZOTUs plateID_all_A16S.trimmed.good_sorted_uniques_ZOTUs.fasta -ampout plateID_all_A16S.trimmed.good_sorted_uniques_ampout.fasta -tabbedout plateID_all_A16S.trimmed.good_sorted_uniques_unoise3.txt -minsize 8

Combine the ZOTUs obtained from sample-wise and plate-wise analysis with a defined set of "prior" sequences into a single fasta file. The resulting file name is standardised to the format:

`plateID_all_A16S.trimmed.good_sorted_uniques_ZOTUs_renamed_SWpriors.fasta`

**Dereplicate the combined ZOTU file:**

    usearch -fastx_uniques plateID_all_A16S.trimmed.good_sorted_uniques_ZOTUs_renamed_SWpriors.fasta -fastaout

The resulting file name is standardised to the format:

`plateID_all_A16S.trimmed.good_sorted_uniques_ZOTUs_renamed_SWpriors_uniq.fasta`

**Map reads to ZOTUs to generate abundances**

Reads are mapped against the ZOTUs using **USEARCH**. Note the termination conditions on the mapping run (**-maxaccepts 0**), this seems to be required to ensure the best match is found and to produce consistent results when adding multiple plates together, as we do later.

    usearch -otutab plateID_all_16S.fasta -ZOTUs plateID_all_A16S.trimmed.good_sorted_uniques_ZOTUs_SWpriors_uniq.fasta -otutabout plateID_all_A16S.trimmed.good_sorted_uniques_PWSW_ZOTUtab_MA0.txt -mapout plateID_all_A16S.trimmed.good_sorted_uniques_PWSW_zmap_MA0.txt -maxaccepts 0 -threads 10

**Replace ZOTU table indexes with ZOTU sequence**

After denoising and mapping the ZOTU tables have an arbitrary ZOTU number as the index, we replace this with the sequence that the arbitrary number represents. These sequences are unique strings and allow tables to be merged etc. downstream easily. They also negate the need to maintain a dictionary of ZOTUs and the sequences they represent.

**Classify and remove flipped sequences**

A final QC step is performed to remove likely erroneous sequences. The ZOTUs are classified, with those that do not align to the Greengenes (DeSantis et al.,2006) 16S database in the correct orientation being removed. Those that need to be "flipped" to a new orientation are likely errors, since we know the reads should be in A2f -- 519r orientation. This step typically removes < 10 ZOTUs from the database.

**A. Classify the seqs against 16S database:**

    mothur "#set.dir(modifynames=F); classify.seqs(fasta=plateID_all_A16S.trimmed.good_sorted_uniques_ZOTUtab_relabelled_MA0.fasta, reference=gg_13_8_99.fasta, taxonomy=gg_13_8_99.gg.tax, cutoff=60, probs=FALSE)"

**B. Use the \*acnos.flip list to remove flipped sequences from the table**

The above produces the final abundance table, with ZOTU sequences as index for each sequencing run. These tables are then combined as below to produce a single dataset for the Australian Microbiome.

### 5. Prepare the single dataset

Now we have a ZOTU abundance table for each plate, with ZOTU's as row and sampleID_plateID as column headers. To prepare this data for ingest into the AM database the following steps are carried out:

1. Each table is converted from short to long format (from rectangular to 3 column, with the following columns: ZOTU, sampleID, Abundance)
2. All of these 3 column tables are concatenated into a single table
3. Controls and samples are split into separate tables
4. Sequencing run ID's (plateIDs) are removed from the column headers and any sample sequenced more than once has the OTUs grouped and abundances summed to give a single abundance per sample
5. A fasta file of unique ZOTU's is created from all ZOTU's in this final table
6. Sequences are classified

**Classify the sequences**

Final taxonomic tables are produced by classifying sequences using a variety of taxonomic databases and classification methodologies, as below:

Databases used for archaeal 16S:

-   Silva 138 (Quast et al., 2013; Yilmaz et al., 2014; Glöckner 2017)

-   Silva 132 (Quast et al., 2013; Yilmaz et al., 2014; Glöckner 2017)

Classifiers are used with the following arguments:

**QIIME2** SKlearn Bayesian classifier:

    --p-n-jobs 10 --p-confidence 0.6 --p-read-orientation same --verbose

**MOTHUR** wang Bayesian classifier:

    cutoff=60, probs=FALSE, processors=8

**USEARCH** usearch_global nearest neighbour classifier:

    -query_cov 1.0 -id 0.8 -maxaccepts 100 -maxrejects 100 -maxhits 1 -strand plus -threads 8

**References**
1. Magoc, T. and Salzberg, S. (2011). FLASH: Fast length adjustment of short reads to improve genome assemblies. Bioinformatics 27(21): 2957-2963.
2. Seqtk available at: <https://github.com/lh3/seqtk> (last accessed 23 Jan 2019).
3. Martin, M. Cutadapt removes adapter sequences from high-throughput sequencing reads [DOI:10.14806/ej.17.1.200] (http://dx.doi.org/10.14806/ej.17.1.200).
4. FASTX available at: (<http://hannonlab.cshl.edu/fastx_toolkit/>) (last accessed 23 Jan 2019).
5. Edgar, R.C., (2010), Search and clustering orders of magnitude faster than BLAST, Bioinformatics 26(19): 2460-2461.
6. Schloss, P.D., et al., (2009). Introducing mothur: Open-source, platform-independent, community-supported software for describing and comparing microbial communities. Appl Environ Microbiol 75(23):7537-7541.
7. DeSantis T.Z., Hugenholtz P., Larsen N., Rojas M., Brodie E.L., Keller K., Huber T., Dalevi D., Hu P., Andersen G.L.(2006) Greengenes, a Chimera-Checked 16S rRNA Gene Database and Workbench Compatible with ARB. Appl. Environ. Microbiol. 72(7): 5069-5072; DOI: 10.1128/AEM.03006-05.
8. Quast C, Pruesse E, Yilmaz P, Gerken J, Schweer T, Yarza P, Peplies J, Glöckner FO (2013) The SILVA ribosomal RNA gene database project: improved data processing and web-based tools. Nucl. Acids Res. 41 (D1): D590-D596.
9. Yilmaz P, Parfrey LW, Yarza P, Gerken J, Pruesse E, Quast C, Schweer T, Peplies J, Ludwig W, Glöckner FO (2014) The SILVA and "All-species Living Tree Project (LTP)" taxonomic frameworks. Nucl. Acids Res. 42:D643-D648.
10. Glöckner FO, Yilmaz P, Quast C, Gerken J, Beccati A, Ciuprina A, Bruns G, Yarza P, Peplies J, Westram R, Ludwig W (2017) 25 years of serving the community with ribosomal RNA gene reference databases and tools. J. Biotechnol. 10:169-176.doi: 10.1016/j.jbiotec.2017.06.1198
11. Bolyen E, Rideout JR, Dillon MR, Bokulich NA, Abnet CC, Al-Ghalith GA, Alexander H, Alm EJ, Arumugam M, Asnicar F, Bai Y, Bisanz JE, Bittinger K, Brejnrod A, Brislawn CJ, Brown CT, Callahan BJ, Caraballo-Rodríguez AM, Chase J, Cope EK, Da Silva R, Diener C, Dorrestein PC, Douglas GM, Durall DM, Duvallet C, Edwardson CF, Ernst M, Estaki M, Fouquier J, Gauglitz JM, Gibbons SM, Gibson DL, Gonzalez A, Gorlick K, Guo J, Hillmann B, Holmes S, Holste H, Huttenhower C, Huttley GA, Janssen S, Jarmusch AK, Jiang L, Kaehler BD, Kang KB, Keefe CR, Keim P, Kelley ST, Knights D, Koester I, Kosciolek T, Kreps J, Langille MGI, Lee J, Ley R, Liu YX, Loftfield E, Lozupone C, Maher M, Marotz C, Martin BD, McDonald D, McIver LJ, Melnik AV, Metcalf JL, Morgan SC, Morton JT, Naimey AT, Navas-Molina JA, Nothias LF, Orchanian SB, Pearson T, Peoples SL, Petras D, Preuss ML, Pruesse E, Rasmussen LB, Rivers A, Robeson MS, Rosenthal P, Segata N, Shaffer M, Shiffer A, Sinha R, Song SJ, Spear JR, Swafford AD, Thompson LR, Torres PJ, Trinh P, Tripathi A, Turnbaugh PJ, Ul-Hasan S, van der Hooft JJJ, Vargas F, Vázquez-Baeza Y, Vogtmann E, von Hippel M, Walters W, Wan Y, Wang M, Warren J, Weber KC, Williamson CHD, Willis AD, Xu ZZ, Zaneveld JR, Zhang Y, Zhu Q, Knight R, and Caporaso JG. (2019) Reproducible, interactive, scalable and extensible microbiome data science using QIIME 2. Nature Biotechnology 37: 852--857.<https://doi.org/10.1038/s41587-019-0209-9>.
