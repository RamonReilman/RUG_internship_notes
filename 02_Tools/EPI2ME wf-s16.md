# Tool: EPI2ME (nextflow) wf-16s

- **Category:** #tool/ASV #tool/classifier
- **Repository / Link:** https://epi2me.nanoporetech.com/epi2me-docs/workflows/wf-16s/#3-classify-reads-taxonomically
- **Primary Paper / Citation:** 

## Overview
This is the taxonomic classification workflow currently used by the department, I will make a note of it to easily find information about it. Since this will be run on a cluster without GUI, a will have to install this workflow via nextflow, tutorial for this will be found below this.
## Installation & Environment
### Requirements
- openjdk=17
First thing to run the workflow on a server without GUI, is nextflow. This can be installed via Conda.
```bash
conda install -c bioconda -c conda-forge nextflow=26.04.6  openjdk=17
```


