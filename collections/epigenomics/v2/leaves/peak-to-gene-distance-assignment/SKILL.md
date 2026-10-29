---
name: peak-to-gene-distance-assignment
description: Use when ENCODE ChIP-seq peak BED files for TRs must be converted into
  gene-level regulatory potential scores via peak-to-TSS distances plus peak signal
  strength weighting.
when_to_use_negative:
- ChIP-seq data for a TR are missing or contain too few targets — the paper warns
  lack of ChIP-seq data or ChIP-seq data with too few targets would mislead TR identification.
- Precomputed regulatory potential scores for the TRs of interest are already available
  on the RePhine google drive resource — load them instead of recomputing.
- The goal is downstream TR-drug response association testing (RePhine regression/elastic-net
  steps) rather than producing the target-inference RP score layer itself.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: Python
  role: Runtime for RP_calculation_modified.py, the modified script that maps each
    ChIP-seq peak to its target gene and sums exp(-(0.5+4d))*m into gene-level RP
    scores
  repo: https://github.com/coexps/RePhine
- name: RePhine
  role: Supplies the modified RP score calculation script and the reference regulatory
    potential scores (more than 1000 ENCODE ChIP-seq datasets) used to validate the
    computed gene x TR matrix
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:ecf6c1f03c035a29624e49bc2755277aae2a4500d27362f17515a4bee67cd224
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - RePhine
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# peak-to-gene-distance-assignment

## Summary

Assigns ChIP-seq peaks of a transcriptional regulator (TR) to target genes by peak-to-TSS distance and computes regulatory potential (RP) scores weighting each peak by its ENCODE signal enrichment value m, providing the quantitative target-inference layer of RePhine.

## When to use

Use when ENCODE ChIP-seq peak BED files for TRs must be converted into gene-level regulatory potential scores via peak-to-TSS distances plus peak signal strength weighting.

## When NOT to use

- ChIP-seq data for a TR are missing or contain too few targets — the paper warns lack of ChIP-seq data or ChIP-seq data with too few targets would mislead TR identification.
- Precomputed regulatory potential scores for the TRs of interest are already available on the RePhine google drive resource — load them instead of recomputing.
- The goal is downstream TR-drug response association testing (RePhine regression/elastic-net steps) rather than producing the target-inference RP score layer itself.

## Inputs

- ENCODE TR ChIP-seq peak BED file(s) retaining each peak's signal enrichment value m
- Gene annotation with transcription start site (TSS) coordinates
- Modified Python RP score calculation script (RP_calculation_modified.py) and reference RP scores from the RePhine repository

## Outputs

- Gene x TR regulatory potential (RP) score matrix written as rp_score_matrix.csv
- Peak-to-target distance assignments (d) underlying each gene's summed RP score

## How to apply

Run RePhine's modified Python script RP_calculation_modified.py, which extends the original RP model beyond peak counts and distances to additionally weight each peak by its ENCODE signal enrichment m. Load the TR's ENCODE ChIP-seq peak set retaining each peak's enrichment value m, plus a gene annotation with transcription start site (TSS) coordinates. For each TR-gene pair, map every ChIP-seq peak to its target gene and compute the peak-to-target distance d. Apply the modified RP formula, summing exp(-(0.5+4d)) * m over all peaks assigned to each gene for each TR, so that both binding proximity and peak signal strength contribute to the score. Assemble the per-TR scores into a gene x TR matrix and write it to rp_score_matrix.csv. Validate by comparing the computed matrix against the reference RP scores distributed with the RePhine repository (more than 1000 ENCODE ChIP-seq datasets on google drive), requiring numerical agreement (output_matches_reference).

## Related tools

- **Python** (Runtime for RP_calculation_modified.py, the modified script that maps each ChIP-seq peak to its target gene and sums exp(-(0.5+4d))*m into gene-level RP scores) — https://github.com/coexps/RePhine
- **RePhine** (Supplies the modified RP score calculation script and the reference regulatory potential scores (more than 1000 ENCODE ChIP-seq datasets) used to validate the computed gene x TR matrix) — https://github.com/coexps/RePhine

## Examples

```
python RP_calculation_modified.py peaks.bed
```

## Evaluation signals

- rp_score_matrix.csv is a gene x TR matrix whose entries equal the sum of exp(-(0.5+4d)) * m over all peaks assigned to that gene for that TR
- Computed RP-score matrix numerically matches the reference RP scores distributed in the RePhine repository (output_matches_reference check)
- Every peak in the input BED file is mapped to a target gene; no peaks are silently dropped before summing
- Distances d are measured from peak coordinates to transcription start site (TSS) coordinates in the gene annotation
- Single-peak genes can be verified by hand: score = exp(-(0.5+4d)) * m for that peak's distance and enrichment

## Limitations

- Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs.
- RePhine identifies drug response-related TRs through the targets, so this RP layer is restricted to cases where reliable target information is available.
- Reference RP scores cover more than 1000 ENCODE ChIP-seq datasets and the repository notes these data 'will be update with new TFs soon' — coverage for a given TR depends on the ENCODE data provided.

## Evidence

- [readme] RP_calculation_modified.py is the modified python script which calculates the RP scores by consider both distance and peak strength. It takes in peak bed files of ChIP-seq data.: [evidence span withheld — anchor status: external_doc]
- [discussion] application of quantitative and informative target inference by considering both ChIP-seq signals and the distances from peaks to the targets: "application of quantitative and informative target inference by considering both ChIP-seq signals and the distances from peaks to the targets"
- [readme] Regulatory potential scores across more than 1000 ChIP-seq data from Encode that are used in RePhine analysis: [evidence span withheld — anchor status: external_doc]
- [discussion] Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs: "Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs"
