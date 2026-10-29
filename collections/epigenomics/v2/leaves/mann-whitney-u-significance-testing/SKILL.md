---
name: mann-whitney-u-significance-testing
description: Use when comparing two independent per-TF score distributions (e.g.,
  AUPR across 150 TFs for two binding-prediction methods) and a non-parametric significance
  assessment of their difference is required.
when_to_use_negative:
- When only a single method's predictions are available and no second per-TF distribution
  exists to compare against
- When the comparison target is nucleosome positioning validated against chemical
  cleavage annotations rather than TF binding site prediction AUPR
- When the data are paired per-TF measurements (the workflow treats the per-TF AUPR
  distributions as independent samples)
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: RoboCOP
  role: Source of probabilistic TF occupancy scores whose per-TF AUPR distributions
    are compared by Mann-Whitney U tests
  repo: https://github.com/HarteminkLab/RoboCOP
- name: COMPETE
  role: Comparison method whose occupancy predictions provide the competing per-TF
    AUPR distribution
- name: MACS2
  role: Peak/footprint identification step of the MACS2+FIMO baseline that generates
    the FIMO-MNase and FIMO-ATAC comparison distributions
- name: FIMO
  role: TF-labeling motif scanning step of the baseline methods whose AUPR distributions
    are tested against RoboCOP's
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:2f8a511904a4554b5e7b68627f47af200bbf1850f97d0df79298cdc4b3ee512c
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Mann-Whitney U significance testing

## Summary

Non-parametric test applied to per-TF AUPR distributions to assess whether RoboCOP configurations (RoboCOP_MNase, RoboCOP_ATAC, RoboCOP_MNase+ATAC) significantly outperform COMPETE variants and the FIMO-MNase/FIMO-ATAC baselines in TF binding site prediction.

## When to use

Use when comparing two independent per-TF score distributions (e.g., AUPR across 150 TFs for two binding-prediction methods) and a non-parametric significance assessment of their difference is required.

## When NOT to use

- When only a single method's predictions are available and no second per-TF distribution exists to compare against
- When the comparison target is nucleosome positioning validated against chemical cleavage annotations rather than TF binding site prediction AUPR
- When the data are paired per-TF measurements (the workflow treats the per-TF AUPR distributions as independent samples)

## Inputs

- Per-TF AUPR vectors for each method (RoboCOP_MNase, RoboCOP_ATAC, RoboCOP_MNase+ATAC, COMPETE variants, FIMO-MNase, FIMO-ATAC) computed over 150 yeast TFs
- ChIP-chip/ChIP-exo/ORGANIC ground truth TF binding annotations
- RoboCOP probabilistic occupancy profiles from MNase-seq (GSE153609) and ATAC-seq (GSE66386) data

## Outputs

- Mann-Whitney U statistics and p-values for pairwise method comparisons
- aupr_comparison.csv containing per-TF and per-method AUPR values with Mann-Whitney U significance results (Figure 5 comparison)

## How to apply

For each method (RoboCOP_MNase, RoboCOP_ATAC, RoboCOP_MNase+ATAC, COMPETE_RoboCOP, COMPETE_Kd/8Kd/16Kd, FIMO-MNase, FIMO-ATAC) rank predicted binding sites by the method's probabilistic occupancy score and compute AUPR against ChIP-chip/ChIP-exo/ORGANIC annotations at single-nucleotide resolution, yielding a per-TF AUPR vector over the 150 yeast TFs. For each pairwise method comparison, run a Mann-Whitney U test on the two per-TF AUPR distributions to test whether one method's ranks are stochastically higher. The rationale is that per-TF AUPR values are not assumed normal, so a rank-based non-parametric test is appropriate for aggregating across TFs. Write the per-TF and per-method AUPR values together with the Mann-Whitney U significance results to aupr_comparison.csv, reproducing the Figure 5 comparison in which RoboCOP fares better than COMPETE, which is in turn significantly better than the FIMO-MNase and FIMO-ATAC baselines.

## Related tools

- **RoboCOP** (Source of probabilistic TF occupancy scores whose per-TF AUPR distributions are compared by Mann-Whitney U tests) — https://github.com/HarteminkLab/RoboCOP
- **COMPETE** (Comparison method whose occupancy predictions provide the competing per-TF AUPR distribution)
- **MACS2** (Peak/footprint identification step of the MACS2+FIMO baseline that generates the FIMO-MNase and FIMO-ATAC comparison distributions)
- **FIMO** (TF-labeling motif scanning step of the baseline methods whose AUPR distributions are tested against RoboCOP's)

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(aupr_robocop_mnase, aupr_compete, alternative='greater')
```

## Evaluation signals

- Per-TF AUPR is computed at single-nucleotide resolution against ChIP-chip/ChIP-exo/ORGANIC annotations for each of the 150 TFs
- aupr_comparison.csv contains both per-TF/per-method AUPR values and the Mann-Whitney U significance results
- Result ordering reproduces Figure 5E: RoboCOP models fare better than COMPETE models, which are significantly better than FIMO-MNase and FIMO-ATAC baselines
- Each method contributes a distribution over the 150 TFs, not a single aggregate score, before testing
- U-test directionality is consistent with the claimed superiority of RoboCOP over COMPETE and FIMO baselines

## Limitations

- No version numbers were stated for any tool (RoboCOP, COMPETE, MACS2, FIMO) used in the benchmark
- MNase-seq gives markedly better RoboCOP performance than ATAC-seq (due to genome-wide fragment distribution and MNase exonuclease activity reflecting DBF sizes), so comparisons involving ATAC-based runs are confounded by input data type
- Accuracy and comprehensiveness of the underlying occupancy profiles improve with deeper sequencing, affecting the AUPR distributions being tested

## Evidence

- [results] Run Mann-Whitney U tests on the per-TF AUPR distributions to assess whether RoboCOP configurations significantly outperform COMPETE and the FIMO baselines.: [evidence span withheld — anchor status: not_found]
- [results] Overall, the RoboCOP models fare better than the COMPETE models, which are in turn significantly better than the FIMO-MNase and FIMO-ATAC baseline models (Figure 5E).: [evidence span withheld — anchor status: not_found]
- [results] compute AUPR against the ChIP-chip/ChIP-exo/ORGANIC annotations at single-nucleotide resolution: [evidence span withheld — anchor status: not_found]
