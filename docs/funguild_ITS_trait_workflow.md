# FUNGuild Trait Analysis of Fungal ITS Amplicons

This document outlines the workflow required to perform metabolic and/or functional trait analysis on amplicon ZOTU sequences of the internal transcribed spacer (ITS) region located between the small and large rRNA subunits.

Analysis is completed using all ZOTUs in the AM ITS1 or ITS2 datasets.

***Note: In this workflow, the use of ITSn represents the ITS region being analysed (e.g., ITS1 or ITS2)***

The workflow consists of the following stages:

1. Sequence analysis and reporting
    - Generate a FUNGuild input file from the taxonomy files generated from the ITSn workflow
    - Use the FUNGuild parser to format the input file
    - Run FUNGuild on the parsed input file
2. Preparation of the single datasets
    - Format the FUNGuild traits file for use in the AM

**Software used**

The following software is used in the workflow:

1. FUNGuild v1.2 (Nguyen et al., 2016)
2. Python3

### 1. Sequence analysis and reporting

**Prepare a FUNGuild readable input file**

For each classification method used, a tab separated FUNGuild formatted input file is prepared using the final taxonomy files produced in **5. Prepare the single dataset** in **fungal_ITS_amplicon_workflow.md**.

**Use the FUNGuild parser to format the input file**

The FUNGuild parser is used to format the input file using the following arguments:

    FUNGuild.py taxa -otu <input_file> -format tsv -column taxonomy -classifier unite

**Run FUNGuild on the parsed input file**

FUNGuild is run on the parsed input file using the following arguments

    FUNGuild.py guild -taxa

### 2. Preparation of the single datasets

**Format the FUNGuild traits file for use in the AM**

For ingest into the AM, only traits represented in the `guild` field are used. For each classification method used, a final tab separated table containing ZOTU sequence, taxonomy, trait and amplicon code is produced for ingesting into the AM database.

**References**
1. Nguyen NH, Song Z, Bates ST, Branco, S, Tedersoo L, Menke J, Schilling JS, Kennedy PG. (2016) FUNGuild: an open annotation tool for parsing fungal community datasets by ecological guild. Fungal Ecology 20:241-248.
