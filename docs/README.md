# Analysis workflow documents
Information relating to the workflows used for processed data products available on the [Australian microbiome data portal](https://data.bioplatforms.com/bpa/otu). 

## Analysis workflow version
Data processing methodology can be determined by date `(dd/mm/yyyy)` or by examining the `Dataset methodology` and/or `Dataset analysis url` value described in the `info.txt` file exported with downloaded data. 

### Version history

#### 2.2.1

**Transition from workflows documented in a single Microsoft word file `(.docx)` to individual markdown `(.md)` files for each analysis**
- Converted workflows in `amplicon_analysis_workflow.docx` to the following markdown documents:
  - archaeal_16S_amplicon_workflow.md
  - bacterial_16S_amplicon_workflow.md
  - eukaryotic_18S_amplicon_workflow.md
  - faprotax_16S_trait_workflow.md
  - fungal_ITS_amplicon_workflow.md
  - funguild_ITS_trait_workflow.md
  - metaxa_ssuRNA_metagenome_workflow.md

- Deleted `amplicon_analysis_workflow.docx` from repository

- No methodological changes to workflows from 2.2.0. However, numerous typographical/organisational changes have been made for clarity. 
    - Corrected information in some commands including:
        - Fixed missing `--plus T` argument in metaxa2 command
        - Deleted unused metaxa2 dependencies
        - Fixed missing arguments `-maxhits 1`, `-strand plus` in usearch global command
        - Fixed missing call to `mothur` in `remove flipped` classification

- **Version control in download packets**
  -   Data analysed with workflow 2.2.1 will contain the following information in the `info.txt` file:
      -   `Dataset methodology=bpaotu_x.x.x__analysis_2.2.1__AM_db_vx.x_xxxxxxxxxxxx.db__AM_data_submit_xxxxxxxxxxxx.tar.gz`

  - Due to the addition of multiple markdown documents and deletion of `amplicon_analysis_workflow.docx`, `Dataset analysis url` in the `info.txt` file will now provide a link to the github tree for the tagged version, as below:
      - `Dataset analysis url=https://github.com/AusMicrobiome/amplicon/tree/2.2.1/docs`

#### 2.2.0
- Data analysed with workflow 2.2.0 will contain the following information in the info.txt file:
  - `Dataset methodology=bpaotu_x.x.x__analysis_2.2.0__AM_db_vx.x_xxxxxxxxxxxx.db__AM_data_submit_xxxxxxxxxxxx.tar.gz`
   
   **Change summary**
   - Added workflow information for final taxonomic classifications with multiple databases and classification methodologies.
   - Text changes for clarity.

#### 2.1.0

- Data analysed with workflow 2.1.0 will contain the following information in the info.txt file:
  - `Dataset methodology=bpaotu_x.x.x__analysis_2.1.0__AM_db_vx.x_xxxxxxxxxxxx.db__AM_data_submit_xxxxxxxxxxxx.tar.gz`
  - `Dataset analysis url=https://github.com/AusMicrobiome/amplicon/raw/2.1.0/docs/amplicon_analysis_workflow.docx`
 - [Download version 2.1.0](https://github.com/AusMicrobiome/amplicon/raw/2.1.0/docs/amplicon_analysis_workflow.docx)
 
   **Change summary**
   - Included workflow for trait analysis of bacterial (27f/519r) and archaeal (2f/519r) ZOTUs using FAPROTAX
   - Included workflow for trait analysis of ITS1 and ITS2 ZOTUs using FUNGuild
   - Typographical modifications
     - Fixed typographical error in ITS `Classify and remove flipped sequences` command for `Classify the seqs against UNITE ITS database`
     - Added missing Metaxa2 references
     - Minor typographical changes for clarity

#### 2.0.0

- Data accessed after `03/08/2021` or with `Dataset methodology=bpaotu_x.x.x_AM_db_v2_xxxxxxxxxxxx.db`
- [Download version 2.0.0](https://github.com/AusMicrobiome/amplicon/raw/2.0.0/docs/amplicon_analysis_workflow.docx)

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
- Data accessed prior to `03/08/2021` or with `Dataset methodology=v1`
- [Download version 1.0.0](https://github.com/AusMicrobiome/amplicon/raw/1.0.0/docs/amplicon_analysis_workflow.docx)
