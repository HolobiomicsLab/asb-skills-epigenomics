---
name: tcga-pancancer-omics-preprocessing
description: Use when raw Firehose GISTIC2.0 lesion output (arm-level or focal, with
  q-values) plus paired TCGA RNASeq expression per cancer type must be transformed
  into iEDGE objects (cn, gep, cisgenes) as RDS for cis driver analysis.
when_to_use_negative:
- Input is already an iEDGE object or RDS file produced by this preprocessing step
  — proceed directly to run_iEDGE.
- The epi-DNA data is not GISTIC2.0-formatted copy-number output (e.g., mutation or
  methylation matrices); other iEDGE converters would be required.
- The goal is a single-cohort analysis rather than uniform preprocessing across the
  19 TCGA cancer types.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: iEDGE
  role: Provides make_GISTIC2 and make_GISTIC2_focal converters and construct_iEDGE
    to build iEDGE objects from GISTIC2.0 and expression data
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Language in which the preprocessing scripts (e.g., scripts_TCGA_pancancer/1.preprocess_all_TCGA.R)
    are written and executed
- name: Biobase
  role: Bioconductor dependency underpinning the iEDGE data structures used to hold
    expression and phenotype data
- name: limma
  role: Imported dependency supporting the differential expression machinery of the
    iEDGE package
- name: GISTIC2.0
  role: Source of the lesion output files (copy-number alteration calls) that the
    converters transform into the cn matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:c34820b8e203a0c14af92180817d2cb69451bc45f7b4b391b938abba75ad2e63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - GISTIC2.0
  - R
  - iEDGE
  - limma
  - rmarkdown
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# tcga-pancancer-omics-preprocessing

## Summary

Converts Firehose GISTIC2.0 copy-number lesion files and TCGA RNASeq expression matrices, per cancer type, into iEDGE-ready objects (cn, gep, cisgenes) suitable for the iEDGE cis/trans driver pipeline, using the make_GISTIC2/make_GISTIC2_focal converters and the preprocessing logic of scripts_TCGA_pancancer/1.preprocess_all_TCGA.R.

## When to use

Use when raw Firehose GISTIC2.0 lesion output (arm-level or focal, with q-values) plus paired TCGA RNASeq expression per cancer type must be transformed into iEDGE objects (cn, gep, cisgenes) as RDS for cis driver analysis.

## When NOT to use

- Input is already an iEDGE object or RDS file produced by this preprocessing step — proceed directly to run_iEDGE.
- The epi-DNA data is not GISTIC2.0-formatted copy-number output (e.g., mutation or methylation matrices); other iEDGE converters would be required.
- The goal is a single-cohort analysis rather than uniform preprocessing across the 19 TCGA cancer types.

## Inputs

- Firehose GISTIC2.0 lesion files (all_lesions with arm-level or focal calls and q-values) per TCGA cancer type
- TCGA RNASeq gene expression matrices per cancer type
- Gene reference list (all_genes) for mapping alteration regions
- Optional sample annotation (gep.pdat) and feature annotation (cn.fdat, gep.fdat)

## Outputs

- Binary copy-number alteration matrix (cn) derived from GISTIC2.0 amplification/deletion lesions
- Aligned gene expression matrix (gep) matching cn samples
- cisgenes list of genes within alteration regions (transgenes may be NA)
- iEDGE dataset object (constructed via construct_iEDGE) stored as RDS per cancer type

## How to apply

For each TCGA cancer type, run the iEDGE converter make_GISTIC2(gistic_in, all_genes, amp_thres_arm, del_thres_arm) on the Firehose GISTIC2.0 lesion file for arm-level alterations, or make_GISTIC2_focal(gistic_in, all_genes, qvalue_focal, all_lesions, f.amp, f.del) when focal events are desired, applying the focal q-value cutoff to select amplification/deletion lesions. Follow the preprocessing logic illustrated by scripts_TCGA_pancancer/1.preprocess_all_TCGA.R to harmonize the resulting binary alteration matrices (cn) with the RNASeq expression matrix (gep), aligning gene identifiers (e.g., HGNC symbols) and samples, and derive cisgenes as genes within the alteration regions. Assemble the data with construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat, cn.pdat = NA, gep.pdat = NA) to produce an iEDGE dataset object. Save each cancer-type object as RDS and load it later with readRDS for run_iEDGE execution. Repeat across all 19 TCGA cancer types so the pan-cancer Rank-1 enrichment analysis can be run on uniformly preprocessed inputs.

## Related tools

- **iEDGE** (Provides make_GISTIC2 and make_GISTIC2_focal converters and construct_iEDGE to build iEDGE objects from GISTIC2.0 and expression data) — https://github.com/montilab/iEDGE
- **R** (Language in which the preprocessing scripts (e.g., scripts_TCGA_pancancer/1.preprocess_all_TCGA.R) are written and executed)
- **Biobase** (Bioconductor dependency underpinning the iEDGE data structures used to hold expression and phenotype data)
- **limma** (Imported dependency supporting the differential expression machinery of the iEDGE package)
- **GISTIC2.0** (Source of the lesion output files (copy-number alteration calls) that the converters transform into the cn matrix)

## Examples

```
dat <- construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat, cn.pdat = NA, gep.pdat = NA)
```

## Evaluation signals

- The GISTIC2.0 lesion file is fully consumed: every focal/amplification/deletion call passing the qvalue_focal cutoff is represented in the binary cn matrix.
- Samples and gene identifiers are congruent: colnames(cn) matches colnames(gep) and cisgenes symbols exist in rownames(gep).
- construct_iEDGE succeeds without error and the returned iEDGE dataset contains cn, gep, and cisgenes components loadable via readRDS.
- Preprocessed outputs are reproducible against the reference logic in scripts_TCGA_pancancer/1.preprocess_all_TCGA.R for the 19 TCGA cancer types.
- Downstream run_iEDGE with fdr.cis.cutoff=0.25, min.group=2, onesided.cis=TRUE runs without preprocessing-induced failures, indicating valid object structure.

## Limitations

- Downstream Sobel mediation validity depends on adequate sample size and strong correlation effects, conditions not met in all TCGA datasets — very small cancer types may yield unreliable preprocessed inputs.
- The choice between arm-level (make_GISTIC2 with amp_thres_arm/del_thres_arm) and focal (make_GISTIC2_focal with qvalue_focal) conversion changes the resulting cn/cisgenes definitions and must match the intended analysis.
- No changelog was found for the iEDGE repository, so converter behavior may drift between package versions.

## Evidence

- [readme] iEDGE is an R-package for performing integrative analysis of (epi-)DNA and gene expression data: "iEDGE is an R-package for performing integrative analysis of (epi-)DNA
and gene expression data"
- [methods] make_GISTIC2_focal converts GISTIC2.0 focal lesions into iEDGE input: "make_GISTIC2_focal(gistic_in, all_genes, qvalue_focal, all_lesions, f.amp, f.del)"
- [methods] make_GISTIC2 handles arm-level GISTIC2.0 calls with amp/del thresholds: "make_GISTIC2(gistic_in, all_genes, amp_thres_arm, del_thres_arm"
- [methods] construct_iEDGE assembles the iEDGE dataset object from cn, gep, and cisgenes: "dat <- construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat, cn.pdat = NA, gep.pdat = NA)"
- [abstract] iEDGE identifies cis and trans gene expression signatures associated with epi-DNA alterations: "iEDGE first identifies the cis and trans gene expression signatures associated with the presence/absence of a particular epi-DNA alteration across samples"
