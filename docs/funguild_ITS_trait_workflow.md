# FUNGuild Trait Analysis of Fungal ITS Amplicons

This document outlines the workflow required to perform metabolic and/or functional trait analysis on amplicon ZOTU sequences of the internal transcribed spacer (ITS) region located between the small and large rRNA subunits.

Analysis is completed using all ZOTUs in the AM ITS1 or ITS2 datasets.

***Note: In this workflow, the use of ITSn represents the ITS region being analysed (e.g., ITS1 or ITS2)***

The workflow consists of the following stages:

1. Sequence analysis and reporting
    - Generate a FUNGuild input file from the taxonomy files generated from the ITSn workflow
    - Use FUNGuild to identify traits
2. Preparation of the single datasets
    - Format the FUNGuild traits file for use in the AM

**Software used**

The following software is used in the workflow:

1. FUNGuild (Nguyen et al., 2016, https://github.com/UMNFuN/FUNGuild) 
2. Python3

### 1. Sequence analysis and reporting

**Prepare a FUNGuild readable input file**

For each classification database (e.g., Unite), a tab separated FUNGuild input file is prepared by combining all unique taxonomy strings identified in **5. Prepare the single dataset** in **fungal_ITS_amplicon_workflow.md**. The FUNGuild input file has the following format:
| OTU_ID | taxonomy |
|-----------|--------------|
| TAX_1 | k__kingdom1;p__phylum1;c__class1;o__order1;f__family1;g__genus1;s__species1 |
| TAX_2 | k__kingdom2;p__phylum2;c__class2;o__order2;f__family2;g__genus2;s__species2 |
| TAX_3 | k__kingdom3;p__phylum3;c__class3;o__order3;f__family3;g__genus3;s__species3 |


**Run FUNGuild **

An output file containing matched traits is generated using the Guilds script with the following arguments:

    Guilds_v1.1.py -otu <input_file> -m -db fungi

### 2. Preparation of the single datasets

**Format the FUNGuild traits file for use in the AM**

For ingest into the AM, traits represented in the `Trophic Mode`, `Guild`, `Growth Morphology`, and `Trait` fields are combined into a single comma separated `traits` column. Pipe `|` separated primary guilds (see [GitHub issue #82]( https://github.com/UMNFuN/FUNGuild/issues/82)) are retained. For each classification method used, a final tab separated table containing sequence identifier (md5sum hash of ZOTU sequence), taxonomy, traits and amplicon code is produced for ingesting into the AM data portal.

**References**
1. Nguyen NH, Song Z, Bates ST, Branco, S, Tedersoo L, Menke J, Schilling JS, Kennedy PG. (2016) FUNGuild: an open annotation tool for parsing fungal community datasets by ecological guild. Fungal Ecology 20:241-248.
