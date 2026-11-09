---
name: hic-contact-map-target-processing
description: Use when raw Hi-C contact matrices must be converted into fixed-resolution
  (5 kb skeleton) binned interaction targets to train a sequence-based chromatin interaction
  predictor such as deepC.
when_to_use_negative:
- When the goal is TAD boundary calling from an existing Hi-C matrix rather than generating
  training targets — a dedicated tool such as TopDom is the appropriate choice
- When Hi-C data are already provided at the target resolution (5 kb skeleton) and
  only model training is required
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: deepC
  role: Target framework whose 5 kb Hi-C skeleton training scheme defines the required
    binning and resolution; includes R scripts and functions for Hi-C data handling
    and downstream analysis
  repo: github.com/rschwess/deepC
- name: pysam
  role: Python module for reading/manipulating SAM/BAM alignment files and tabix interfaces
    during Hi-C target extraction
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Genomic interval arithmetic for binning interactions onto the 5 kb skeleton;
    installed alongside pysam via pip or bioconda
- name: cowplot
  role: R plotting package used for downstream analysis and visualisation of Hi-C
    data and deepC predictions
  repo: https://github.com/wilkelab/cowplot
- name: TopDom
  role: Alternative deterministic method for identifying topological domains from
    Hi-C data; relevant when TAD-level outputs rather than model training targets
    are needed
  repo: https://github.com/HenrikBengtsson/TopDom
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:3bba4a9506dd1dc32beb609c6bae0ed1ee38949c613fef7e38536cd3a386d007
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - TopDom
  - cowplot
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# hic-contact-map-target-processing

## Summary

Processes Hi-C contact data (e.g. Rao et al. GM12878/K562, GEO GSE63525) into 5 kb-resolution binned skeleton target tracks used to train deepC chromatin interaction models, using pysam/pybedtools for genomic interval handling and R scripts for Hi-C downstream analysis.

## When to use

Use when raw Hi-C contact matrices must be converted into fixed-resolution (5 kb skeleton) binned interaction targets to train a sequence-based chromatin interaction predictor such as deepC.

## When NOT to use

- When the goal is TAD boundary calling from an existing Hi-C matrix rather than generating training targets — a dedicated tool such as TopDom is the appropriate choice
- When Hi-C data are already provided at the target resolution (5 kb skeleton) and only model training is required

## Inputs

- Hi-C contact data (e.g. Rao et al., GEO GSE63525) for cell types such as GM12878 and K562
- Genomic interval files (BED) handled via pybedtools
- SAM/BAM or tabix-indexed alignment data handled via pysam

## Outputs

- 5 kb-resolution Hi-C skeleton target tracks used as training labels for the deepC interaction network
- Trained model checkpoints in HDF5 format with training logs recording epochs and loss

## How to apply

Start from publicly deposited Hi-C data such as Rao et al. (GEO GSE63525) for the cell type of interest (e.g. GM12878, K562). Use pysam and pybedtools (installable via pip or bioconda) to manipulate genomic intervals and extract contact information, binning interactions onto a 5 kb-resolution skeleton so that each target window corresponds to the deepC model's megabase-scale input region. Use the repository's R scripts (R 3.4.4+, tidyverse, cowplot, rtracklayer) and custom Hi-C handling functions for downstream processing, track export, and visualisation of the binned contact targets. The resulting skeleton targets become the training labels for the chromatin interaction network, which is trained with GPU support for several hours to days depending on dataset size and epochs, with checkpoints saved in HDF5 format alongside epoch/loss training logs. Correctness is judged by verifying checkpoint files load as valid HDF5 models and that the saved network's parameter count is within tolerance of the expected ~60M parameters.

## Related tools

- **deepC** (Target framework whose 5 kb Hi-C skeleton training scheme defines the required binning and resolution; includes R scripts and functions for Hi-C data handling and downstream analysis) — github.com/rschwess/deepC
- **pysam** (Python module for reading/manipulating SAM/BAM alignment files and tabix interfaces during Hi-C target extraction) — https://github.com/pysam-developers/pysam
- **pybedtools** (Genomic interval arithmetic for binning interactions onto the 5 kb skeleton; installed alongside pysam via pip or bioconda)
- **cowplot** (R plotting package used for downstream analysis and visualisation of Hi-C data and deepC predictions) — https://github.com/wilkelab/cowplot
- **TopDom** (Alternative deterministic method for identifying topological domains from Hi-C data; relevant when TAD-level outputs rather than model training targets are needed) — https://github.com/HenrikBengtsson/TopDom

## Examples

```
pip install pybedtools pysam
```

## Evaluation signals

- Confirm each target window is binned at the expected 5 kb resolution and matches the Hi-C skeleton layout of the source contact map
- Verify trained model checkpoint files exist and load as valid HDF5 model files
- Check the saved interaction network's parameter count is within tolerance of the expected ~60M parameters
- Confirm training logs record epochs and loss and that the training script runs end to end on GPU
- Cross-check binned targets against the source cell type (e.g. GM12878 or K562) metadata of the Rao et al. GSE63525 data

## Limitations

- deepC was developed under tensorflow v1.8 and supports only v1.8, 1.14 and 2.1 — other versions are untested, and GPU support is essential for training
- Training the interaction network on the Hi-C targets requires running with GPU support for several hours up to days depending on dataset size and epochs aimed for
- Hi-C data are low depth and low resolution relative to 3C-based assays; deepC compensates via pre-seeding but target quality still depends on the source Hi-C

## Evidence

- [other] 5 kb Hi-C skeleton targets derived from Rao et al. Hi-C (GEO GSE63525) for GM12878 and K562: [evidence span withheld — anchor status: not_found]
