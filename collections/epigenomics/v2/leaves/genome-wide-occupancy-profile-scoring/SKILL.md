---
name: genome-wide-occupancy-profile-scoring
description: Use when comparing TF binding site predictions from chromatin occupancy
  models (RoboCOP, COMPETE, MACS2+FIMO/FIMO baselines) against ChIP-chip, ChIP-exo,
  or ORGANIC annotations to quantify and rank method accuracy.
when_to_use_negative:
- The task is generating occupancy profiles themselves rather than scoring/benchmarking
  predictions against ChIP ground truth
- No ChIP-chip, ChIP-exo, or ORGANIC ground truth annotations are available for the
  TFs being evaluated
- Predictions are from organisms or assays other than the yeast MNase-seq/ATAC-seq
  chromatin accessibility context supported by RoboCOP/COMPETE
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: RoboCOP
  role: Multivariate HMM/state space model producing the genome-wide probabilistic
    occupancy scores of nucleosomes and TFs that are ranked and evaluated by AUPR
  repo: https://github.com/HarteminkLab/RoboCOP
- name: COMPETE
  role: Earlier sequence-based method computing a probabilistic landscape of DBF occupancy,
    used as a comparison method in the AUPR benchmark
- name: MACS2
  role: Peak or footprint identification used to construct the baseline predictions
    for the MNase-seq and ATAC-seq comparison arms
- name: FIMO
  role: TF-labeling by motif scanning within MACS2 peaks/footprints to build the FIMO-MNase
    and FIMO-ATAC baseline predictions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:97e5371b1146a31b4b0244fb45d87b9d94dbd0ad6066485e854b8b62f7e89301
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - COMPETE
  - FIMO
  - MACS2
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# genome-wide-occupancy-profile-scoring

## Summary

Benchmark genome-wide probabilistic TF binding site predictions (e.g. from RoboCOP, COMPETE, or MACS2+FIMO baselines) by ranking predicted sites per TF and computing AUPR against ChIP-chip/ChIP-exo/ORGANIC ground truth annotations, as done for the RoboCOP yeast chromatin occupancy profiles.

## When to use

Use when comparing TF binding site predictions from chromatin occupancy models (RoboCOP, COMPETE, MACS2+FIMO/FIMO baselines) against ChIP-chip, ChIP-exo, or ORGANIC annotations to quantify and rank method accuracy.

## When NOT to use

- The task is generating occupancy profiles themselves rather than scoring/benchmarking predictions against ChIP ground truth
- No ChIP-chip, ChIP-exo, or ORGANIC ground truth annotations are available for the TFs being evaluated
- Predictions are from organisms or assays other than the yeast MNase-seq/ATAC-seq chromatin accessibility context supported by RoboCOP/COMPETE

## Inputs

- Genome-wide probabilistic occupancy profiles of nucleosomes and 150 TFs from RoboCOP (MNase-seq, ATAC-seq/GSE66386, or joint inputs)
- COMPETE probabilistic occupancy predictions (RoboCOP_Kd, Kd/8Kd, 16Kd variants)
- MACS2 peaks/footprints with FIMO motif scan TF labels (MNase-seq and ATAC-seq baselines)
- ChIP-chip, ChIP-exo, and ORGANIC ground truth TF binding annotations

## Outputs

- Per-TF, per-method AUPR values at single-nucleotide resolution (aupr_comparison.csv)
- Mann-Whitney U significance results comparing per-TF AUPR distributions between methods
- Aggregated AUPR distributions reproducing the Figure 5E method comparison

## How to apply

Load the probabilistic occupancy profiles for each method configuration (RoboCOP_MNase, RoboCOP_ATAC, RoboCOP_MNase+ATAC, COMPETE variants, and MACS2+FIMO baselines built for both MNase-seq and ATAC-seq inputs). For each of the 150 TFs, rank predicted binding sites by the method's probabilistic score and compute AUPR against ChIP-chip/ChIP-exo/ORGANIC ground truth annotations at single-nucleotide resolution. Build the MACS2+FIMO baselines by identifying peaks or footprints genome-wide with MACS2 and labeling TF binding sites within those peaks via FIMO motif scanning. Aggregate per-TF AUPR distributions across TFs for each method, then run Mann-Whitney U tests on the per-TF AUPR distributions to test whether higher-fidelity models (e.g. RoboCOP configurations) significantly outperform baselines. The expected outcome, per the paper, is RoboCOP > COMPETE >> FIMO-MNase/FIMO-ATAC baselines (Figure 5E).

## Related tools

- **RoboCOP** (Multivariate HMM/state space model producing the genome-wide probabilistic occupancy scores of nucleosomes and TFs that are ranked and evaluated by AUPR) — https://github.com/HarteminkLab/RoboCOP
- **COMPETE** (Earlier sequence-based method computing a probabilistic landscape of DBF occupancy, used as a comparison method in the AUPR benchmark)
- **MACS2** (Peak or footprint identification used to construct the baseline predictions for the MNase-seq and ATAC-seq comparison arms)
- **FIMO** (TF-labeling by motif scanning within MACS2 peaks/footprints to build the FIMO-MNase and FIMO-ATAC baseline predictions)

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git && cd RoboCOP/robocop && bash gccCompile
```

## Evaluation signals

- Per-TF AUPR is computed at single-nucleotide resolution against ChIP-chip/ChIP-exo/ORGANIC annotations, not against peak-level calls
- Method ranking reproduces the paper's ordering: RoboCOP models > COMPETE models >> FIMO-MNase and FIMO-ATAC baselines (Figure 5E)
- Mann-Whitney U tests report significant differences between RoboCOP and COMPETE/FIMO per-TF AUPR distributions across the 150 TFs
- aupr_comparison.csv contains AUPR values for all method configurations (RoboCOP_MNase, RoboCOP_ATAC, RoboCOP_MNase+ATAC, COMPETE variants, FIMO-MNase, FIMO-ATAC) and all TFs

## Limitations

- No version numbers are stated for any of the tools (RoboCOP, COMPETE, MACS2, FIMO), so exact reproducibility of the benchmark requires environment pinning
- MNase-seq inputs give markedly better RoboCOP performance than ATAC-seq for two primary reasons (genome-wide fragment distribution and MNase exonuclease activity reflecting DBF sizes), so absolute AUPR values are assay-dependent
- Accuracy of chromatin occupancy profiles improves with deeper sequencing; shallow data will depress AUPR for all methods
- Prior information about DBF transition probabilities was not incorporated in this version and is deferred to future work

## Evidence

- [results] Overall, the RoboCOP models fare better than the COMPETE models, which are in turn significantly better than the FIMO-MNase and FIMO-ATAC baseline models (Figure 5E).: [evidence span withheld — anchor status: not_found]
