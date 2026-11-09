---
name: macs2-fimo-baseline-pipeline-construction
description: Use when benchmarking a chromatin occupancy or TF binding site prediction
  method and a simple peak/footprint-then-motif-scan baseline is needed for comparison
  against ChIP-chip/ChIP-exo/ORGANIC ground truth annotations.
when_to_use_negative:
- When the goal is the most accurate occupancy profile rather than a benchmark baseline
  — the paper shows RoboCOP with MNase- or ATAC-seq data performs notably better than
  the MACS2+FIMO approach
- When no ground truth binding annotations (ChIP-chip/ChIP-exo/ORGANIC) are available
  to compute per-TF AUPR for the comparison
- When single-nucleotide-resolution probabilistic occupancy across hundreds of DBFs
  is the target output rather than discrete motif-labeled peaks
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: MACS2
  role: Identifies peaks or footprints genome-wide from MNase-seq or ATAC-seq data
    as the first step of the baseline pipeline
- name: FIMO
  role: Labels TF binding sites within the MACS2-defined peaks by motif scanning,
    producing the FIMO-MNase and FIMO-ATAC baselines
- name: RoboCOP
  role: Provides the probabilistic occupancy framework against whose per-TF AUPR distributions
    the MACS2+FIMO baselines are compared (Mann-Whitney U tests)
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:82a0690074217158a8b630de9db97b417037a44d62da564678a1717f9f9b63df
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - COMPETE
  - FIMO
  - MACS2
  - RoboCOP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# macs2-fimo-baseline-pipeline-construction

## Summary

Constructs the MACS2+FIMO baseline for TF binding site prediction: identify peaks or footprints genome-wide with MACS2 from MNase-seq or ATAC-seq input, then label TF binding sites within those regions via FIMO motif scanning, so that its per-TF AUPR can be compared against probabilistic models such as RoboCOP and COMPETE (Figure 5E).

## When to use

Use when benchmarking a chromatin occupancy or TF binding site prediction method and a simple peak/footprint-then-motif-scan baseline is needed for comparison against ChIP-chip/ChIP-exo/ORGANIC ground truth annotations.

## When NOT to use

- When the goal is the most accurate occupancy profile rather than a benchmark baseline — the paper shows RoboCOP with MNase- or ATAC-seq data performs notably better than the MACS2+FIMO approach
- When no ground truth binding annotations (ChIP-chip/ChIP-exo/ORGANIC) are available to compute per-TF AUPR for the comparison
- When single-nucleotide-resolution probabilistic occupancy across hundreds of DBFs is the target output rather than discrete motif-labeled peaks

## Inputs

- MNase-seq dataset (e.g. GSE153609 / SRP041314) for FIMO-MNase peak calling
- ATAC-seq dataset (GSE66386) for FIMO-ATAC peak calling
- Reference genome for FIMO motif scanning
- Position weight matrix / motif file for the 150 Saccharomyces cerevisiae TFs
- ChIP-chip, ChIP-exo, and ORGANIC ground truth TF binding annotations

## Outputs

- MACS2 peak/footprint calls genome-wide, one set per assay (MNase-seq and ATAC-seq)
- FIMO-labeled TF binding sites within MACS2 peaks, ranked by probabilistic score
- Per-TF AUPR values for the FIMO-MNase and FIMO-ATAC baselines (contributing to aupr_comparison.csv)

## How to apply

Run MACS2 peak (or footprint) identification genome-wide separately on the MNase-seq and ATAC-seq datasets (ATAC-seq data from GSE66386), producing one set of accessible regions per assay. Within those MACS2-defined regions, scan TF motifs with FIMO to label candidate binding sites for each of the 150 TFs. Rank the FIMO-labeled sites by the method's probabilistic score and compute AUPR against ChIP-chip/ChIP-exo/ORGANIC annotations at single-nucleotide resolution, yielding the FIMO-MNase and FIMO-ATAC baselines. Aggregate per-TF AUPR distributions across the 150 TFs and test differences against RoboCOP and COMPETE configurations with Mann-Whitney U tests. The expected outcome, per the paper, is that the FIMO baselines perform significantly worse than both COMPETE and RoboCOP, so they serve as the lower reference points in the aupr_comparison.csv benchmark table.

## Related tools

- **MACS2** (Identifies peaks or footprints genome-wide from MNase-seq or ATAC-seq data as the first step of the baseline pipeline)
- **FIMO** (Labels TF binding sites within the MACS2-defined peaks by motif scanning, producing the FIMO-MNase and FIMO-ATAC baselines)
- **RoboCOP** (Provides the probabilistic occupancy framework against whose per-TF AUPR distributions the MACS2+FIMO baselines are compared (Mann-Whitney U tests)) — https://github.com/HarteminkLab/RoboCOP

## Evaluation signals

- Both assay-specific baselines are produced: FIMO-MNase from MNase-seq peaks and FIMO-ATAC from ATAC-seq peaks (GSE66386)
- Per-TF AUPR is computed at single-nucleotide resolution against ChIP-chip/ChIP-exo/ORGANIC annotations for the 150 TFs
- AUPR distributions for the baselines should fall significantly below both the COMPETE variants and the RoboCOP configurations by Mann-Whitney U test
- Per-method AUPR values and significance results are written to aupr_comparison.csv, reproducing the Figure 5E comparison

## Limitations

- The baseline is a deliberately weaker reference: the paper reports RoboCOP models fare better than COMPETE, which in turn are significantly better than the FIMO-MNase and FIMO-ATAC baselines
- No version numbers are stated for MACS2 or FIMO, so exact reproducibility of the baseline requires pinning tool versions externally
- Baseline quality depends on peak-calling sensitivity and motif-scan stringency; the paper does not quantify MACS2/FIMO parameters used

## Evidence

- [other] Overall, the RoboCOP models fare better than the COMPETE models, which are in turn significantly better than the FIMO-MNase and FIMO-ATAC baseline models (Figure 5E).: [evidence span withheld — anchor status: not_found]
- [other] Build the MACS2+FIMO baseline by identifying peaks or footprints genome-wide with MACS2 and labeling TF binding sites within those peaks via FIMO motif scanning, for both MNase-seq and ATAC-seq inputs.: [evidence span withheld — anchor status: not_found]
