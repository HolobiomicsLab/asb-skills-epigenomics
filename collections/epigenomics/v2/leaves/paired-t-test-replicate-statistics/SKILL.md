---
name: paired-t-test-replicate-statistics
description: Use when eCLIP/skipper reproducible_enriched_windows tables exist for
  matched WT and mutant replicates and you need to quantify region-proportion shifts
  (CDS, deep introns) or test differential enrichment (e.g., tRNA loci) with replicate-level
  statistics.
when_to_use_negative:
- Only a single replicate exists per condition, so no replicate pairing is possible
  and a t-test is underpowered/invalid
- Input is already a differential-enrichment table with per-window p-values; re-testing
  proportions would double-count statistics
- CLIP data lack the matched input samples Skipper requires to build its beta-binomial
  model, so no reproducible_enriched_windows exist to aggregate
edam_operation: http://edamontology.org/operation_0396
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Skipper
  role: Produces the reproducible_enriched_windows tables and feature_type_top annotations
    whose per-replicate aggregates feed the paired t-tests
  repo: https://github.com/YeoLab/skipper
- name: Snakemake
  role: Drives the Skipper end-to-end pipeline (version 9.12.0) that generates the
    window tables used in the statistical comparison
  repo: https://github.com/YeoLab/skipper
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:8d86add935ee76d630a189f4e8741d449ac761dfd5024b95b400bf0dd27b82b6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  repo_url: https://github.com/YeoLab/skipper
  tools:
  - Skipper
  - Snakemake
  edam_operation: http://edamontology.org/operation_0396
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# paired-t-test-replicate-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical comparison skill that pairs matched eCLIP replicates (e.g., WT vs mutant RPS19 R62W/R101H) and applies paired t-tests on per-replicate Skipper-derived enrichment values, such as reproducible-enriched-window feature proportions and per-replicate tRNA enrichment, to test whether mutants shift the RNA interactome.

## When to use

Use when eCLIP/skipper reproducible_enriched_windows tables exist for matched WT and mutant replicates and you need to quantify region-proportion shifts (CDS, deep introns) or test differential enrichment (e.g., tRNA loci) with replicate-level statistics.

## When NOT to use

- Only a single replicate exists per condition, so no replicate pairing is possible and a t-test is underpowered/invalid
- Input is already a differential-enrichment table with per-window p-values; re-testing proportions would double-count statistics
- CLIP data lack the matched input samples Skipper requires to build its beta-binomial model, so no reproducible_enriched_windows exist to aggregate

## Inputs

- reproducible_enriched_windows tables (Skipper output) for WT, R62W, and R101H RPS19 eCLIP replicates
- GFF-derived feature annotations (feature_type_top per window: CDS, deep introns, UTRs, tRNA loci)
- Replicate mapping (matched WT/mutant pairs)

## Outputs

- Per-sample fraction table of reproducible enriched windows by feature_type_top
- figure7e_feature_proportions.pdf (WT vs mutant region proportion comparison)
- figure7f_trna_enrichment.pdf (per-replicate tRNA enrichment with paired t-test significance annotations)
- Paired t-test statistics and p-values for WT vs R62W and WT vs R101H tRNA enrichment

## How to apply

Load the reproducible_enriched_windows tables Skipper produced for WT, R62W, and R101H replicates, and aggregate windows by feature_type_top within each sample to compute the fraction of reproducible enriched windows per transcript region. Quantify the WT-to-mutant shift in region proportions (expected: CDS 33%->45%, deep introns 20%->7%, p < 2.2e-16) and render it as figure7e_feature_proportions.pdf. For locus-specific testing, subset windows mapping to annotated tRNA loci, compute per-replicate tRNA enrichment values, and pair each mutant replicate with its matched WT replicate for a paired t-test, which exploits replicate-to-replicate pairing rather than pooling. Render per-figure tRNA enrichment with paired-test significance annotations (figure7f_trna_enrichment.pdf) and accept the analysis only if feature proportions reproduce the reported shifts and both mutants show significant tRNA enrichment p-values relative to WT.

## Related tools

- **Skipper** (Produces the reproducible_enriched_windows tables and feature_type_top annotations whose per-replicate aggregates feed the paired t-tests) — https://github.com/YeoLab/skipper
- **Snakemake** (Drives the Skipper end-to-end pipeline (version 9.12.0) that generates the window tables used in the statistical comparison) — https://github.com/YeoLab/skipper

## Examples

```
from scipy.stats import ttest_rel; t, p = ttest_rel(mut_r62w_trna_enrichment, wt_trna_enrichment)
```

## Evaluation signals

- Computed CDS window fraction rises from 33% (WT) to 45% (mutants) and deep intronic fraction falls from 20% to 7%
- Region-proportion shift significance reaches p < 2.2e-16 when comparing WT to mutant feature distributions
- Paired t-tests yield significant tRNA enrichment p-values for BOTH mutants (R62W and R101H) relative to matched WT replicates
- Per-sample feature fractions sum to ~1 across feature_type_top categories within each replicate
- Figures figure7e_feature_proportions.pdf and figure7f_trna_enrichment.pdf render with the paired-test annotations

## Limitations

- eCLIP data alone do not answer whether observed binding changes arise from differences in complex formation, localization, or binding preference
- RPS19 follow-up relied on expressing variant ORFs in HEK293T cells, which may not reflect the true disease state
- Skipper requires matched input samples to construct its beta-binomial model, so replicate statistics are only possible for datasets with such controls

## Evidence

- [results] Paired t-tests across matched WT/mutant replicates: [evidence span withheld — anchor status: not_found]
- [results] Region proportion shift finding: [evidence span withheld — anchor status: not_found]
