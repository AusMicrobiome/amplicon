# Analysis workflow documents
Information relating to the workflows used for processed data products available on the [Australian microbiome data portal](https://data.bioplatforms.com/bpa/otu). 

## Analysis workflow version
Data processing methodology can be determined by date `(dd/mm/yyyy)` or by examining the `Dataset methodology` value described in the `info.txt` file exported with downloaded data. 

### Version history

#### 2.0.0

- Data accessed after `03/08/2021` or with `Dataset methodology=bpaotu_x.x.x_AM_db_v2_xxxxxxxxxxxx.db`: [Download version 2.0.0](https://github.com/AusMicrobiome/amplicon/raw/2.0.0/docs/amplicon_analysis_workflow.docx)

  **Change summary**
  - Amplicon sequence variant (ZOTU) calling is performed on a per sample and per sequencing plate basis with ZOTUs from each method combined.
  - Amplicon sequence variant (ZOTU) workflows allow for the inclusion of user defined "prior" sequences.
  - Workflow for archaeal 16S (A2f-519r) amplicon analysis amended to include primer removal.
  - Minimum and maximum sequence length restrictions have been removed from bacterial 16S (27f – 519r) sequence analysis.
  - Minimum number of representative sequences (UNOISE3 -minsize ) in ZOTU calling has been changed from 4 to 8.
  - Defined file naming for samples downloaded from NCBI.
  - Amplicon sequence classification for archaeal and bacterial 16S, 18Sv4 and 18Sv9 is performed using QIIME2 sklearn Bayesian classifier using SILVA 138.
  - ITS1 and ITS2 classification is performed using UNITE SH v8.

#### 1.0.0
- Data accessed prior to `03/08/2021` or with `Dataset methodology=v1`: [Download version 1.0.0](https://github.com/AusMicrobiome/amplicon/raw/1.0.0/docs/amplicon_analysis_workflow.docx)
