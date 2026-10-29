---
name: chip-annotation-ground-truth-mapping
description: Use when a probabilistic TF occupancy method needs quantitative validation
  against ChIP-chip/ChIP-exo/ORGANIC ground-truth binding annotations, e.g. comparing
  RoboCOP runs to COMPETE variants and FIMO baselines across many TFs.
when_to_use_negative:
- The task is generating occupancy profiles themselves (run RoboCOP or COMPETE directly)
  rather than evaluating existing predictions against ChIP annotations
- Ground-truth annotations are chemical-cleavage-based nucleosome positions rather
  than TF binding annotations (ChIP-chip/ChIP-exo/ORGANIC) — use the nucleosome-positioning
  validation route instead
- Only a single TF and a single method are under study, where a distributional Mann-Whitney
  U comparison across per-TF AUPRs is not meaningful
tools:
- name: RoboCOP
  role: Source of the probabilistic TF occupancy predictions being validated (MNase,
    ATAC, and joint runs)
  repo: https://github.com/HarteminkLab/RoboCOP
- name: COMPETE
  role: Sequence/affinity-based occupancy predictor used as a comparison model in
    the AUPR benchmark
- name: MACS2
  role: Peak or footprint identification step that, followed by FIMO TF-labeling,
    defines the baseline prediction methods
- name: FIMO
  role: TF-labeling via motif scanning within MACS2 peaks/footprints to build the
    FIMO-MNase and FIMO-ATAC baselines
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:ebacf7d9b8a29ca14dc291d06e818b0671e25648fc228e51e2a9f51af35bfb62
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# chip-annotation-ground-truth-mapping

## Summary

Benchmarks genome-wide TF binding site predictions from chromatin occupancy models (RoboCOP, COMPETE) and MACS2+FIMO baselines against high-precision ChIP-chip, ChIP-exo, and ORGANIC binding annotations, ranking per-TF performance by AUPR at single-nucleotide resolution.

## When to use

Use when a probabilistic TF occupancy method needs quantitative validation against ChIP-chip/ChIP-exo/ORGANIC ground-truth binding annotations, e.g. comparing RoboCOP runs to COMPETE variants and FIMO baselines across many TFs.

## When NOT to use

- The task is generating occupancy profiles themselves (run RoboCOP or COMPETE directly) rather than evaluating existing predictions against ChIP annotations
- Ground-truth annotations are chemical-cleavage-based nucleosome positions rather than TF binding annotations (ChIP-chip/ChIP-exo/ORGANIC) — use the nucleosome-positioning validation route instead
- Only a single TF and a single method are under study, where a distributional Mann-Whitney U comparison across per-TF AUPRs is not meaningful

## Inputs

- RoboCOP probabilistic TF occupancy profiles computed from MNase-seq data
- RoboCOP probabilistic TF occupancy profiles computed from ATAC-seq data (GSE66386)
- RoboCOP profiles computed jointly from MNase-seq and ATAC-seq data
- COMPETE occupancy predictions (RoboCOP-informed and Kd/8Kd/16Kd variants)
- FIMO-MNase and FIMO-ATAC baseline predictions (MACS2 peak/footprint calls plus FIMO motif scanning)
- ChIP-chip, ChIP-exo, and ORGANIC TF binding annotations as ground truth

## Outputs

- Per-TF, per-method AUPR values computed at single-nucleotide resolution
- Mann-Whitney U significance results comparing per-TF AUPR distributions across methods
- aupr_comparison.csv reproducing the Figure 5 comparison

## How to apply

Load the predicted occupancy profiles for each method variant (RoboCOP_MNase, RoboCOP_ATAC, RoboCOP_MNase+ATAC, COMPETE variants, FIMO-MNase and FIMO-ATAC baselines built from MACS2 peaks/footprints followed by FIMO motif labeling) and the ChIP-chip/ChIP-exo/ORGANIC annotations. For each of the 150 TFs, rank predicted binding sites by the method's probabilistic score and compute AUPR against the ChIP annotations at single-nucleotide resolution. Aggregate per-TF AUPR distributions per method and run Mann-Whitney U tests on these distributions to test whether RoboCOP configurations significantly outperform COMPETE and the FIMO baselines. The expected outcome, per the paper, is RoboCOP > COMPETE > FIMO-MNase/FIMO-ATAC (Figure 5E), so the benchmark is designed to reproduce that ordering as a sanity check.

## Related tools

- **RoboCOP** (Source of the probabilistic TF occupancy predictions being validated (MNase, ATAC, and joint runs)) — https://github.com/HarteminkLab/RoboCOP
- **COMPETE** (Sequence/affinity-based occupancy predictor used as a comparison model in the AUPR benchmark)
- **MACS2** (Peak or footprint identification step that, followed by FIMO TF-labeling, defines the baseline prediction methods)
- **FIMO** (TF-labeling via motif scanning within MACS2 peaks/footprints to build the FIMO-MNase and FIMO-ATAC baselines)

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git
```

## Evaluation signals

- Per-TF AUPR is computed at single-nucleotide resolution against ChIP-chip/ChIP-exo/ORGANIC annotations for all 150 TFs
- Method-level ordering matches the paper: RoboCOP models outperform COMPETE models, which significantly outperform FIMO-MNase and FIMO-ATAC baselines
- Mann-Whitney U tests on per-TF AUPR distributions report significant differences between RoboCOP and baseline configurations
- Benchmark output is written as aupr_comparison.csv containing per-TF and per-method AUPR values plus significance results

## Limitations

- Markedly better performance is observed when using MNase-seq than ATAC-seq, due to genome-wide fragment distributions and MNase exonuclease activity reflecting DBF sizes, so ATAC-based runs benchmark lower
- The accuracy and comprehensiveness of chromatin occupancy profiles improve with deeper sequencing, so sequencing depth affects benchmark outcomes
- No version numbers are stated for MACS2, FIMO, COMPETE, or RoboCOP in the described comparison, hindering exact reproduction

## Evidence

- [intro] We validate its nucleosome positioning predictions using high-precision annotations resulting from a chemical cleavage method (24): "We validate its nucleosome positioning predictions using high-precision annotations resulting from a chemical cleavage method (24)"
- [discussion] RoboCOP with MNase- or ATAC-seq performs notably better than MACS2 peak/footprint identification followed by FIMO TF-labeling: "RoboCOP with MNase- or ATAC-seq data performs notably better than alternative approaches that use peak or footprint identification with MACS2 (36) followed by TF-labeling with FIMO (35)"
- [readme] README: RoboCOP description: "Example configuration file provided in analysis directory ([config_example.ini]"
