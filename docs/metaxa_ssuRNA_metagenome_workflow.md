# Metaxa Identification of Small Subuint Ribosomal RNA in Metagenomes

This document outlines the workflow required to identify small subunit ribosomal (SSU rRNA) from metagenomic studies for inclusion in the Australian microbiome database.

Analysis is completed on a per sequencing run (sequencing plate) basis. The workflow consists of the following stages:

1. **Sequence analysis and reporting**
   - Identify paired end reads with identity to SSU rRNA using Metaxa2 (v2.2.3)
   - Summarise taxonomic assignments
2. **Preparation of the single dataset**
   - Merge tables into a single table

**Software used**

The following software is used in the workflow:

1. Metaxa2 v2.2.3 (Bengtsson-Palme et al., 2015)
   - Note: Metaxa2 requires dependencies to be installed. The following are used:
	   - HMMER version 3.3.2 (http://hmmer.janelia.org/software)
	   - BLAST+ (v2.12.0) (https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/)
	   - MAFFT v7.481 (http://mafft.cbrc.jp/alignment/software/)
2. Python3

### 1. Sequence analysis and reporting

**Taxonomic Analysis of Sequences**

Metaxa2 is run with the following arguments:

    metaxa2 -1 sampleID_R1.fastq.gz -2 sampleID_R2.fastq.gz -o sampleID_out -f q --plus T --date T --cpu 20 --table T

**Reporting Taxonomy Predictions**

The Metaxa2 Taxonomy Traversal tool is used to report the taxonomic predictions at specific cutoffs that approximately correspond to different nodes of the taxonomic tree (e.g., kingdoms, phyla, classes, orders, families, genera, species, subspecies). The output of this analysis also includes the total abundance of each unique taxa observed.

The input for the taxonomy traversal tool is the sampleID_taxonomy.txt file obtained during Metaxa2 analysis, and is invoked using the following command:

    metaxa2_ttt -i sampleID_taxonomy.txt -o sampleID_taxonomy.txt

### 2.  Prepare the single dataset

Now we have a taxonomy/abundance table for each sample read pair, with the taxonomy and abundance represented as tab separated columns. To prepare this data for ingest into the AM database the following steps are carried out using python3:

1. Using the level 7 taxonomy file(s), sampleID's are added to each row as a tab separated column
2. All 3 column tables are concatenated into a single table
3. Reads classified as unknown, mitochondria or chloroplast are removed
4. Any sample sequenced more than once are grouped by sampleID and taxonomy string with abundances summed to give a single abundance

**References**
1. Bengtsson-Palme J, Hartmann M, Eriksson KM, Pal C, Thorell K, Larsson DGJ, Nilsson RH: Metaxa2: Improved Identification and Taxonomic Classification of Small and Large Subunit rRNA in Metagenomic Data (2015) Molecular Ecology Resources. doi: 10.1111/1755-0998.12399
