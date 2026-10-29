---
name: motif-instance-enumeration-across-cutoffs
description: Use when identifying motif instances (TFBSs) from CNN motif detector
  outputs and you need an objective, statistically grounded cutoff instead of a fixed/subjective
  signal threshold.
when_to_use_negative:
- You only need raw CNN motif detector scores without statistically selected TFBS
  instances — the predict.py step alone suffices.
- Your motif detector is not from DESSO's CNN framework and lacks per-instance signal
  scores across a cutoff grid, so Omega(M, lambda) sets cannot be enumerated.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: DESSO
  role: Framework whose predict.py motif-instance outputs feed the binomial model;
    integrates the binomial distribution to optimize TFBS identification instead of
    a subjective cutoff
  repo: https://github.com/OSU-BMBL/DESSO
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:dfcba9bcbe29cb0b44ae162a3582f93f152c459d98fa44c246058399049b6656
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/OSU-BMBL/DESSO
  repo_url: https://github.com/OSU-BMBL/DESSO
  tools:
  - DESSO
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# motif-instance-enumeration-across-cutoffs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs DESSO's binomial-distribution statistical model that replaces subjective motif signal cutoffs: for each motif detector, TFBS candidate sets are enumerated at varying signal cutoffs, binomial P-values are computed against a GC-matched background, and the cutoff with the minimum P-value selects the optimal motif instances.

## When to use

Use when identifying motif instances (TFBSs) from CNN motif detector outputs and you need an objective, statistically grounded cutoff instead of a fixed/subjective signal threshold.

## When NOT to use

- You only need raw CNN motif detector scores without statistically selected TFBS instances — the predict.py step alone suffices.
- Your motif detector is not from DESSO's CNN framework and lacks per-instance signal scores across a cutoff grid, so Omega(M, lambda) sets cannot be enumerated.

## Inputs

- encode_101_background: GC-matched background set of 500,000 101-bp genomic bins (placed in data/)
- DESSO predict.py motif-instance outputs generated with start_cutoff 0.01, end_cutoff 1, step_cutoff 0.03
- Trained CNN motif detectors from DESSO (output/encode_101/gc_match/<dataset>/Seq/CNN)

## Outputs

- binomial_pvalues.tsv: per-motif, per-cutoff binomial P-value table with pass/fail retention decisions per motif
- pvalue_validation_report.tsv: deviations of computed P-values from an independent binomial-CDF reference implementation
- Optimal motif instance sets Omega(M, lambda_min) corresponding to the minimum binomial P-value per motif

## How to apply

Load the GC-matched background set of 500,000 101-bp bins (encode_101_background, unpacked into data/) and parse the motif-instance outputs produced by DESSO's predict.py run with start_cutoff 0.01, end_cutoff 1, step_cutoff 0.03. For each motif M and cutoff lambda, build the instance set Omega(M, lambda) of peak sequences whose predicted binding sites pass that cutoff. Estimate per-motif occurrence probabilities from the 500,000 background bins and compute exact binomial CDF P-values (formula 5) for each Omega(M, lambda), selecting the candidates corresponding to the minimum P-value as the optimal motif instances. Apply the retention filters — motifs kept only if P < 1e-4, more than 5 motif instances, and at least 3 positions with information content > 1 — and record per-motif, per-cutoff P-values and pass/fail decisions. Validate computed P-values against an independent binomial-CDF reference implementation at fixed lambda values, recording deviations.

## Related tools

- **DESSO** (Framework whose predict.py motif-instance outputs feed the binomial model; integrates the binomial distribution to optimize TFBS identification instead of a subjective cutoff) — https://github.com/OSU-BMBL/DESSO

## Examples

```
python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- Each retained motif has binomial P < 1e-4, more than 5 motif instances, and at least 3 positions with information content > 1.
- The selected cutoff per motif corresponds to the minimum binomial P-value across the cutoff grid (0.01 to 1, step 0.03).
- Computed P-values agree with an independent binomial-CDF reference implementation at fixed lambda values, with deviations logged in pvalue_validation_report.tsv.
- Occurrence probabilities are estimated from the full 500,000 GC-matched background bins, not from the peak set.
- binomial_pvalues.tsv contains an entry for every motif and every cutoff lambda examined, with an explicit pass/fail retention decision.

## Limitations

- The binomial model assumes motif occurrences are independent and identically distributed according to background occurrence probabilities; GC-matching the background is required for validity.
- The cutoff grid (start_cutoff 0.01, end_cutoff 1, step_cutoff 0.03) limits resolution; the true optimum may fall between grid points.
- Retention thresholds (P < 1e-4, >5 instances, >=3 positions with IC > 1) may discard weak but real motifs, e.g. those with few instances in the dataset.

## Evidence

- [other] DESSO's binomial-based statistical model determines the optimal motif instances: [evidence span withheld — anchor status: not_found]
