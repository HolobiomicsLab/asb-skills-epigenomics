---
name: chip-seq-peak-signal-integration
description: Use when you have TR ChIP-seq peak bed files with signal enrichment values
  and gene TSS annotations and need quantitative TR-target scores (RP scores) rather
  than binary target lists.
when_to_use_negative:
- ChIP-seq data lacks reliable target information or contains too few targets — the
  paper notes such data 'would mislead the identification of TRs'
- You only need a binary TR-target list rather than quantitative, signal-weighted
  regulation strengths
- Peak files lack signal enrichment values (m), making the modified peak-strength
  weighting inapplicable
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0204
tools:
- name: RePhine
  role: Provides RP_calculation_modified.py, the modified Python script that computes
    RP scores considering both distance and peak strength; reference RP scores for
    >1000 ENCODE ChIP-seq datasets are distributed alongside
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Runtime for executing RP_calculation_modified.py on peak bed files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:6182ee5c728e5fc4ab3ac2c75c2bba52ce2bf8719d0748693c745fc6890b0fa3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - RePhine
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# chip-seq-peak-signal-integration

## Summary

Compute regulatory-potential (RP) scores from ENCODE TR ChIP-seq peaks using RePhine's modified model, which weights each peak-to-target distance contribution by the peak's ENCODE signal enrichment (m) in addition to peak count and distance, yielding a gene x TR RP-score matrix for downstream drug-response regulator identification.

## When to use

Use when you have TR ChIP-seq peak bed files with signal enrichment values and gene TSS annotations and need quantitative TR-target scores (RP scores) rather than binary target lists.

## When NOT to use

- ChIP-seq data lacks reliable target information or contains too few targets — the paper notes such data 'would mislead the identification of TRs'
- You only need a binary TR-target list rather than quantitative, signal-weighted regulation strengths
- Peak files lack signal enrichment values (m), making the modified peak-strength weighting inapplicable

## Inputs

- ENCODE TR ChIP-seq peak bed files (peaks with signal enrichment value m)
- Gene annotation with transcription start site (TSS) coordinates
- Reference RP scores from the RePhine repository (Google Drive resource) for validation

## Outputs

- Gene x TR regulatory-potential score matrix written to rp_score_matrix.csv
- Per-TR quantitative target-gene binding scores integrating distance and peak strength

## How to apply

Load each ENCODE TR ChIP-seq peak set retaining the enrichment value m per peak, plus gene annotation with transcription start site coordinates. For every TR-gene pair, map each ChIP-seq peak to its target gene and compute the peak-to-target distance d. Apply the modified RP formula, summing exp(-(0.5+4d)) * m over all peaks assigned to each gene, so both binding proximity and peak signal strength contribute — this extends the original RP model, which used only peak counts and distances. Assemble the per-TR scores into a gene x TR matrix written to rp_score_matrix.csv, then compare against the reference RP scores distributed with RePhine (Google Drive, >1000 ENCODE ChIP-seq datasets) to confirm numerical agreement. Use the repository's RP_calculation_modified.py rather than re-implementing, since it encodes the exact modified weighting.

## Related tools

- **RePhine** (Provides RP_calculation_modified.py, the modified Python script that computes RP scores considering both distance and peak strength; reference RP scores for >1000 ENCODE ChIP-seq datasets are distributed alongside) — https://github.com/coexps/RePhine
- **Python** (Runtime for executing RP_calculation_modified.py on peak bed files)

## Examples

```
python RP_calculation_modified.py  # input: ENCODE ChIP-seq peak bed files (with enrichment m) + TSS annotation; output: gene x TR rp_score_matrix.csv
```

## Evaluation signals

- Generated RP-score matrix must match the reference RP scores deposited in the RePhine repository (output_matches_reference check)
- Output is a complete gene x TR matrix persisted as rp_score_matrix.csv
- Every ChIP-seq peak is mapped to a target gene with a computed distance d before scoring
- Scores reflect the modified formula exp(-(0.5+4d)) * m, i.e. peaks with higher enrichment m contribute proportionally more at equal distance

## Limitations

- Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead TR identification
- Method is restricted to cases where reliable target information is available
- Reference RP scores on Google Drive are stated to 'be update with new TFs soon', so coverage may be incomplete

## Evidence

- [readme] RP_calculation_modified.py is the modified python script which calculates the RP scores by consider both distance and peak strength.: [evidence span withheld — anchor status: external_doc]
- [readme] Regulatory potential scores across more than 1000 ChIP-seq data from Encode: [evidence span withheld — anchor status: external_doc]
- [discussion] RePhine task 3: quantitative target inference using ChIP-seq signals and peak-to-target distances: "application of quantitative and informative target inference by considering both ChIP-seq signals and the distances from peaks to the targets"
