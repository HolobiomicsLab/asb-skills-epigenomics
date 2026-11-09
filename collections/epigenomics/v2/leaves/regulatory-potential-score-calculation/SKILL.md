---
name: regulatory-potential-score-calculation
description: Use when ChIP-seq peak BED files with signal enrichment values and gene/TSS
  annotations must be converted into quantitative TR-target regulatory potential scores
  for target inference.
when_to_use_negative:
- No ChIP-seq data are available, or the ChIP-seq data have too few targets, which
  would mislead TR identification
- Reliable target information for the TR of interest is unavailable, since RePhine
  identifies drug-response TRs through their targets
- The goal is direct TR-drug association from mRNA levels only, without ChIP-seq-based
  target inference
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: RePhine
  role: Source of the modified RP score calculation script (RP_calculation_modified.py)
    and reference RP scores used for validation
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Execution environment for RP_calculation_modified.py, which computes RP scores
    from ChIP-seq peak BED files
- name: R
  role: Downstream environment (RePhine R package with glmnet, MASS, lmtest, ppcor,
    psych, ROCR) that consumes the RP scores for TR identification
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:80fbfc82bb953d1867c8da85adb5120d82f3a9846ca51651d949b2cc38574cdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - R
  - RePhine
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# regulatory-potential-score-calculation

## Summary

Calculates regulatory-potential (RP) scores that quantify how strongly a transcriptional regulator binds each target gene, using a modified model that weights ChIP-seq peaks by both peak-to-TSS distance and peak signal strength (enrichment m). These gene x TR RP scores are the target-inference input RePhine needs for drug-response TR identification.

## When to use

Use when ChIP-seq peak BED files with signal enrichment values and gene/TSS annotations must be converted into quantitative TR-target regulatory potential scores for target inference.

## When NOT to use

- No ChIP-seq data are available, or the ChIP-seq data have too few targets, which would mislead TR identification
- Reliable target information for the TR of interest is unavailable, since RePhine identifies drug-response TRs through their targets
- The goal is direct TR-drug association from mRNA levels only, without ChIP-seq-based target inference

## Inputs

- ENCODE ChIP-seq peak BED files for each TR, including peak signal enrichment values (m)
- Gene annotation with transcription start site (TSS) coordinates
- Reference RP scores from the RePhine repository (Google Drive) for validation
- RP_calculation_modified.py script from the RePhine repository

## Outputs

- Gene x TR regulatory potential score matrix (e.g. rp_score_matrix.csv)
- RP-score matrix numerically matching the reference RP scores in the RePhine repository

## How to apply

Download the reference RP scores and the modified Python script RP_calculation_modified.py from the RePhine repository (https://github.com/coexps/RePhine). Load ENCODE TR ChIP-seq peak sets retaining each peak's signal enrichment value m, plus gene annotation with transcription start site coordinates. For each TR-gene pair, map every ChIP-seq peak to its target gene, compute the peak-to-target distance d, and apply the modified RP formula summing exp(-(0.5+4d)) * m over all peaks assigned to each gene, so both binding proximity and peak signal strength contribute — unlike the original model that considered only peak counts and distances. Assemble the per-TR scores into a gene x TR matrix (e.g. rp_score_matrix.csv). Validate by comparing the computed matrix against the reference RP scores distributed in the repository (reference scores cover more than 1000 ENCODE ChIP-seq datasets via the linked Google Drive).

## Related tools

- **RePhine** (Source of the modified RP score calculation script (RP_calculation_modified.py) and reference RP scores used for validation) — https://github.com/coexps/RePhine
- **Python** (Execution environment for RP_calculation_modified.py, which computes RP scores from ChIP-seq peak BED files)
- **R** (Downstream environment (RePhine R package with glmnet, MASS, lmtest, ppcor, psych, ROCR) that consumes the RP scores for TR identification) — https://github.com/coexps/RePhine

## Examples

```
python RP_calculation_modified.py --peak bed_files/ --tss gene_annotation.bed
```

## Evaluation signals

- Output_matches_reference check: the generated RP-score matrix must numerically agree with the reference RP scores distributed in the RePhine repository
- Each TR-gene entry equals the sum of exp(-(0.5+4d)) * m over all peaks assigned to that gene, so scores are non-negative and bounded by contributions of nearby, high-enrichment peaks
- Peaks with greater distance d from the TSS contribute exponentially less weight, so genes with only distant peaks receive near-zero scores
- Matrix dimensionality matches the gene annotation x set of input ChIP-seq TRs, with one column per TR

## Limitations

- Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs
- RePhine is restricted to cases where reliable target information is available
- The reference RP score set covers more than 1000 ENCODE ChIP-seq datasets and is updated with new TFs over time, so re-verification may be needed as data are refreshed

## Evidence

- [readme] RP_calculation_modified.py is the modified python script which calculates the RP scores by consider both distance and peak strength. It takes in peak bed files of ChIP-seq data.: [evidence span withheld — anchor status: external_doc]
- [readme] Regulatory potential scores across more than 1000 ChIP-seq data from Encode that are used in RePhine analysis are available on google drive.: [evidence span withheld — anchor status: external_doc]
- [discussion] application of quantitative and informative target inference by considering both ChIP-seq signals and the distances from peaks to the targets: "application of quantitative and informative target inference by considering both ChIP-seq signals and the distances from peaks to the targets"
- [discussion] modified Python scripts of RP score calculation, and R code for simulation and application for CCLE data are also available: "modified Python scripts of RP score calculation, and R code for simulation and application for CCLE data are also available"
