# FAPROTAX Trait Analysis of Bacteria and Archaea Small Subunit Ribosomal RNA

This document outlines the workflow required to perform metabolic and/or functional trait analysis on amplicon sequences for Bacteria (B16S: 27f--519r) and Archaea (A16S: A2f-519r) for inclusion in the Australian microbiome database.

Analysis is completed using all ZOTUs in the AM archaeal 16S or bacterial 16S datasets. 

The workflow consists of the following stages:

1. **Sequence analysis and reporting**
    - Perform taxonomic assignments using SILVA 132
    - Identify unique taxonomic strings and prepare a FAPROTAX readable table
    - Generate FAPROTAX output file
2. **Preparation of the single datasets**
    - Build a ZOTU, taxonomy and trait table

### **Software used**

The following software is used in the workflow:

1. QIIME2 (Bolyen et al., 2019)
2. Mothur (Schloss, et al., 2009)
3. USEARCH (Edgar 2010)
4. FAPROTAX v1.2.4 (Louca et al., 2016)
5. Python3

### 1. Sequence analysis and reporting

**Classify the sequences**

Unique ZOTU sequences are classified to provide taxonomies relative to the Silva 132 database (Quast et al., 2013; Yilmaz et al 2014; Glöckner 2017). Sequence classification is performed using the following classifiers with the following arguments:

**QIIME2** SKlearn Bayesian classifier:

    --p-confidence 0.6 --p-read-orientation same

**MOTHUR** wang Bayesian classifier:

    cutoff=60, probs=FALSE, processors=8

**USEARCH** usearch_global nearest neighbour classifier:

    -query_cov 1.0 -id 0.8 -maxaccepts 100 -maxrejects 100 -maxhits 1 -strand plus -threads 8

**Prepare a FAPROTAX readable input file**

Unique taxonomic classification strings are used to construct a tab separated FAPROTAX readable input file in the following format:

| #OTU ID | All_samples | taxonomy          |
| ------- | ----------- | --------          |
| 0       | 1           | <taxonomy string> |
| 1       | 1           | <taxonomy string> |
| 2       | 1           | <taxonomy string> |


**Generate FAPROTAX output file**

The FAPROTAX script collapse_table.py is used to produce the faprotax output file using the following arguments:

    -c "#" -d taxonomy --omit_columns 0 --column_names_are_in last_comment_line -v

### 2. Preparation of the single datasets

**Build a ZOTU, taxonomy and trait table**

The FAPROTAX output report is used to build a table of all traits associated with a specific taxonomy string. This table is combined with the full Silva 132 classification table to produce a table containing the ZOTU sequence, taxonomy and trait.

As FAPROTAX trait analysis is optimised for SILVA 132 taxonomy, and multiple taxonomy databases/classifiers are used to generate taxonomic tables for use in the AM data portal the trait information for alternate taxonomies is inferred from the SILVA 132 classification. In this case, the ZOTU sequence is used to combine/merge the SILVA 132 based trait table generated using a specific classification method with the alternate taxonomy classified using the corresponding methodology. Using the alternate taxonomy SILVA 138 as an example:

- SILVA 138 classified by SKlearn is merged with SILVA 132 classified by SKlearn
- SILVA 138 classified by wang is merged with SILVA 132 classified by wang
- SILVA 138 classified by nearest neighbour is merged with SILVA 132 classified by nearest neighbour

In cases where a ZOTU has trait information generated in the SILVA 132 taxonomy, but the alternate taxonomy is unclassified at the phylum/supergroup level, the trait information is removed. The final tab separated table contains ZOTU sequence, taxonomy, trait and amplicon code.

**References**
1. Bolyen E, Rideout JR, Dillon MR, Bokulich NA, Abnet CC, Al-Ghalith GA, Alexander H, Alm EJ, Arumugam M, Asnicar F, Bai Y, Bisanz JE, Bittinger K, Brejnrod A, Brislawn CJ, Brown CT, Callahan BJ, Caraballo-Rodríguez AM, Chase J, Cope EK, Da Silva R, Diener C, Dorrestein PC, Douglas GM, Durall DM, Duvallet C, Edwardson CF, Ernst M, Estaki M, Fouquier J, Gauglitz JM, Gibbons SM, Gibson DL, Gonzalez A, Gorlick K, Guo J, Hillmann B, Holmes S, Holste H, Huttenhower C, Huttley GA, Janssen S, Jarmusch AK, Jiang L, Kaehler BD, Kang KB, Keefe CR, Keim P, Kelley ST, Knights D, Koester I, Kosciolek T, Kreps J, Langille MGI, Lee J, Ley R, Liu YX, Loftfield E, Lozupone C, Maher M, Marotz C, Martin BD, McDonald D, McIver LJ, Melnik AV, Metcalf JL, Morgan SC, Morton JT, Naimey AT, Navas-Molina JA, Nothias LF, Orchanian SB, Pearson T, Peoples SL, Petras D, Preuss ML, Pruesse E, Rasmussen LB, Rivers A, Robeson MS, Rosenthal P, Segata N, Shaffer M, Shiffer A, Sinha R, Song SJ, Spear JR, Swafford AD, Thompson LR, Torres PJ, Trinh P, Tripathi A, Turnbaugh PJ, Ul-Hasan S, van der Hooft JJJ, Vargas F, Vázquez-Baeza Y, Vogtmann E, von Hippel M, Walters W, Wan Y, Wang M, Warren J, Weber KC, Williamson CHD, Willis AD, Xu ZZ, Zaneveld JR, Zhang Y, Zhu Q, Knight R, and Caporaso JG. (2019) Reproducible, interactive, scalable and extensible microbiome data science using QIIME 2. Nature Biotechnology 37: 852--857.<https://doi.org/10.1038/s41587-019-0209-9>.
2. Schloss, P.D., et al., (2009). Introducing mothur: Open-source, platform-independent, community-supported software for describing and comparing microbial communities. Appl Environ Microbiol 75(23):7537-7541.
3. Edgar, R.C., (2010), Search and clustering orders of magnitude faster than BLAST, Bioinformatics 26(19): 2460-2461.
4. Quast C, Pruesse E, Yilmaz P, Gerken J, Schweer T, Yarza P, Peplies J, Glöckner FO (2013) The SILVA ribosomal RNA gene database project: improved data processing and web-based tools. Nucl. Acids Res. 41 (D1): D590-D596.
5. Yilmaz P, Parfrey LW, Yarza P, Gerken J, Pruesse E, Quast C, Schweer T, Peplies J, Ludwig W, Glöckner FO (2014) The SILVA and "All-species Living Tree Project (LTP)" taxonomic frameworks. Nucl. Acids Res. 42:D643-D648.
6. Glöckner FO, Yilmaz P, Quast C, Gerken J, Beccati A, Ciuprina A, Bruns G, Yarza P, Peplies J, Westram R, Ludwig W (2017) 25 years of serving the community with ribosomal RNA gene reference databases and tools. J. Biotechnol. 10:169-176.doi: 10.1016/j.jbiotec.2017.06.1198
7. Louca, S., Parfrey, L.W., Doebeli, M. (2016) Decoupling function and taxonomy in the global ocean microbiome. Science 353:1272-1277.
