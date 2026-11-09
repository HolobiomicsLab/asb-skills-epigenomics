---
name: tf-binding-site-precision-recall-evaluation
description: Use when you need to quantitatively compare genome-wide TF binding site
  predictors (e.g., RoboCOP vs COMPETE vs FIMO baselines) using probabilistic occupancy
  scores and ChIP-based ground-truth binding annotations.
when_to_use_negative:
- The goal is validating nucleosome positioning rather than TF binding sites — the
  article validates nucleosome predictions with chemical cleavage method annotations,
  not ChIP-based TF annotations.
- No ground-truth TF binding annotations (ChIP-chip/ChIP-exo/ORGANIC) are available
  for the organism/TFs, making AUPR at single-nucleotide resolution impossible.
- Only a single predictor is available with no competing methods — the skill's core
  is a comparative AUPR benchmark across RoboCOP, COMPETE, and MACS2+FIMO baselines.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3169
tools:
- name: RoboCOP
  role: Generates genome-wide probabilistic occupancy scores for nucleosomes and 150
    TFs from MNase-seq and/or ATAC-seq data; its predictions are ranked and scored
    by AUPR
  repo: https://github.com/HarteminkLab/RoboCOP
- name: COMPETE
  role: Baseline occupancy predictor producing a probabilistic landscape of DBF occupancy
    compared against RoboCOP in the AUPR benchmark
- name: MACS2
  role: Peak or footprint identification used to construct the alternative baseline
    models evaluated alongside RoboCOP
- name: FIMO
  role: TF-labeling of MACS2 peaks/footprints via motif scanning to build the FIMO-MNase
    and FIMO-ATAC baseline predictions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:71b723e2ae2e970249e22ad25549e4d78603f1b468d841ff744e2f391035c1f9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - COMPETE
  - FIMO
  - MACS2
  - RoboCOP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tf-binding-site-precision-recall-evaluation

## Summary

Benchmarking protocol that ranks predicted TF binding sites from RoboCOP (MNase-seq, ATAC-seq, or joint), COMPETE variants, and MACS2+FIMO baselines by their probabilistic scores and scores them as per-TF AUPR at single-nucleotide resolution against ChIP-chip/ChIP-exo/ORGANIC ground-truth annotations, with Mann-Whitney U tests on the per-TF AUPR distributions.

## When to use

Use when you need to quantitatively compare genome-wide TF binding site predictors (e.g., RoboCOP vs COMPETE vs FIMO baselines) using probabilistic occupancy scores and ChIP-based ground-truth binding annotations.

## When NOT to use

- The goal is validating nucleosome positioning rather than TF binding sites — the article validates nucleosome predictions with chemical cleavage method annotations, not ChIP-based TF annotations.
- No ground-truth TF binding annotations (ChIP-chip/ChIP-exo/ORGANIC) are available for the organism/TFs, making AUPR at single-nucleotide resolution impossible.
- Only a single predictor is available with no competing methods — the skill's core is a comparative AUPR benchmark across RoboCOP, COMPETE, and MACS2+FIMO baselines.

## Inputs

- RoboCOP probabilistic TF occupancy profiles from MNase-seq data
- RoboCOP probabilistic TF occupancy profiles from ATAC-seq data (GSE66386)
- RoboCOP probabilistic TF occupancy profiles computed from MNase-seq and ATAC-seq jointly
- COMPETE occupancy predictions (COMPETE_RoboCOP, COMPETE_Kd/8Kd/16Kd)
- MACS2 peaks/footprints with FIMO motif-scan TF labels for MNase-seq and ATAC-seq inputs
- ChIP-chip/ChIP-exo/ORGANIC ground-truth TF binding annotations
- Motif/PWM set for FIMO scanning of the 150 yeast TFs

## Outputs

- Per-TF, per-method AUPR values computed at single-nucleotide resolution across 150 TFs
- Mann-Whitney U test significance results comparing per-TF AUPR distributions between methods
- aupr_comparison.csv reproducing the Figure 5E method comparison

## How to apply

Load the RoboCOP probabilistic TF occupancy profiles computed from MNase-seq, ATAC-seq (GEO accession GSE66386), and both data types jointly, along with COMPETE occupancy predictions (COMPETE_RoboCOP, COMPETE_Kd/8Kd/16Kd) and the ChIP-chip/ChIP-exo/ORGANIC ground-truth binding annotations. Build the MACS2+FIMO baselines by identifying peaks or footprints genome-wide with MACS2 and labeling TF binding sites within those peaks via FIMO motif scanning, separately for MNase-seq and ATAC-seq inputs. For each method (RoboCOP_MNase, RoboCOP_ATAC, RoboCOP_MNase+ATAC, COMPETE variants, FIMO-MNase, FIMO-ATAC) and each of the 150 TFs, rank predicted binding sites by the method's probabilistic score and compute AUPR against the annotations at single-nucleotide resolution. Aggregate per-TF AUPR distributions across the 150 TFs per method, run Mann-Whitney U tests to assess whether RoboCOP configurations significantly outperform COMPETE and the FIMO baselines, and write per-TF/per-method AUPR values plus significance results to aupr_comparison.csv, reproducing the Figure 5 comparison.

## Related tools

- **RoboCOP** (Generates genome-wide probabilistic occupancy scores for nucleosomes and 150 TFs from MNase-seq and/or ATAC-seq data; its predictions are ranked and scored by AUPR) — https://github.com/HarteminkLab/RoboCOP
- **COMPETE** (Baseline occupancy predictor producing a probabilistic landscape of DBF occupancy compared against RoboCOP in the AUPR benchmark)
- **MACS2** (Peak or footprint identification used to construct the alternative baseline models evaluated alongside RoboCOP)
- **FIMO** (TF-labeling of MACS2 peaks/footprints via motif scanning to build the FIMO-MNase and FIMO-ATAC baseline predictions)

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git; cd RoboCOP/robocop && bash gccCompile
```

## Evaluation signals

- Expected ordering of mean per-TF AUPR: RoboCOP models > COMPETE models > FIMO-MNase/FIMO-ATAC baselines (as in Figure 5E).
- Per-TF AUPR is computed for all 150 TFs for every method, at single-nucleotide resolution against ChIP-chip/ChIP-exo/ORGANIC annotations.
- Mann-Whitney U tests on per-TF AUPR distributions yield significant p-values for RoboCOP configurations over COMPETE and over the FIMO baselines.
- aupr_comparison.csv contains per-TF and per-method AUPR rows plus Mann-Whitney U significance columns.
- RoboCOP_MNase should outperform RoboCOP_ATAC on per-TF AUPR, consistent with the paper's observation of markedly better MNase-seq performance.

## Limitations

- MNase-seq gives markedly better RoboCOP performance than ATAC-seq, so results from the ATAC-seq variant should be interpreted in that light.
- No version numbers are stated for any benchmarked tool (RoboCOP, COMPETE, MACS2, FIMO), complicating exact reproducibility.
- The FIMO baselines depend entirely on MACS2 peak/footprint calling quality followed by motif scanning, limiting their sensitivity to bound sites outside called peaks.
- Prior information about DBF transition probabilities was not incorporated in these predictions (deferred to future work), which may cap achievable TF binding prediction accuracy.

## Evidence
