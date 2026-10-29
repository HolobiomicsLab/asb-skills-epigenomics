---
name: histone-mark-odds-ratio-computation
description: Use when you have two sets of genomic intervals (e.g., true vs predicted
  regulatory indels) plus interval annotations for chromatin interactions or histone
  marks and need per-feature enrichment odds ratios.
when_to_use_negative:
- When the goal is to predict nc-sindel regulatory scores rather than quantify their
  regional enrichment — use the TIVAN-indel train.py/predict.py pipeline instead
- When annotations lack interval coordinates (e.g., only per-variant scores), since
  the contingency-table logic requires interval-overlap of sindels against regulatory
  features
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: TIVAN-indel
  role: Source of predicted nc-sindel functional scores whose enrichment is compared
    against true cis-eQTL labels
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: Python
  role: Builds 2x2 contingency tables with interval-overlap logic in pandas/numpy
    and runs scipy.stats.fisher_exact
- name: R
  role: Alternative fisher.test implementation for odds-ratio computation
- name: matplotlib
  role: Generates boxplot comparisons of odds ratios across features and label sets
    (version 3.4.3)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad060
    title: 'TIVAN-indel: a computational framework for annotating and predicting non-coding
      regulatory small insertions and deletions'
  source_content_hash: sha256:7dc8d072d5cec3837010f08ca26db89930272f49ae82b42ada1c261a821984d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/TIVAN-indel
  tools:
  - Python
  - R
  - TIVAN-indel
  - matplotlib
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# histone-mark-odds-ratio-computation

## Summary

Quantifies enrichment or depletion of noncoding small indel (nc-sindel) label sets — true GTEx cis-eQTL labels and TIVAN-indel predicted scores — in regulatory regions (3DIV PE/PP promoter-centered interactions, Roadmap open chromatin, and seven histone marks) by building 2x2 contingency tables and applying Fisher's exact test to obtain odds ratios.

## When to use

Use when you have two sets of genomic intervals (e.g., true vs predicted regulatory indels) plus interval annotations for chromatin interactions or histone marks and need per-feature enrichment odds ratios.

## When NOT to use

- When the goal is to predict nc-sindel regulatory scores rather than quantify their regional enrichment — use the TIVAN-indel train.py/predict.py pipeline instead
- When annotations lack interval coordinates (e.g., only per-variant scores), since the contingency-table logic requires interval-overlap of sindels against regulatory features

## Inputs

- True noncoding-sindel label sets derived from GTEx cis-eQTL analyses across 44 tissues
- TIVAN-indel predicted functional scores (output.txt from predict.py)
- 3DIV PE/PP promoter-centered chromatin interaction intervals
- Roadmap open-chromatin interval annotations
- Roadmap histone-mark interval annotations for seven marks (including H3K36me3, H3K27me3, H3K9me3)

## Outputs

- enrichment_odds_ratios.csv containing an odds-ratio entry for every regulatory feature x label-set combination
- Boxplot comparisons of odds ratios across features and label sets (matplotlib==3.4.3, Figures 7-8 style)

## How to apply

Load the true nc-sindel labels (from GTEx cis-eQTL analyses across 44 tissues) and the predicted labels (from TIVAN-indel scores via 'python predict.py --path ../test --model ../res/model. --output ../res/output.txt') together with 3DIV PE/PP promoter-centered interaction intervals and Roadmap open-chromatin plus seven histone-mark annotations. For each regulatory feature (PE/PP interactions, open chromatin, H3K36me3, H3K27me3, H3K9me3, etc.) and each label set, build 2x2 contingency tables of sindel overlap vs non-overlap using interval-overlap logic in pandas/numpy. Apply Fisher's exact test (scipy.stats.fisher_exact or R fisher.test) to each table to obtain odds ratios. Summarize median odds ratios per feature and label set in enrichment_odds_ratios.csv and generate boxplot comparisons with matplotlib==3.4.3 (Figures 7-8 style). The decision point for correctness is reproducing the reported pattern: PE/PP OR>1, OR<1 depletion for repressive marks H3K27me3 and H3K9me3, and a higher H3K36me3 median OR for predicted labels (~3.71) than true labels (~2.62).

## Related tools

- **TIVAN-indel** (Source of predicted nc-sindel functional scores whose enrichment is compared against true cis-eQTL labels) — https://github.com/lichen-lab/TIVAN-indel
- **Python** (Builds 2x2 contingency tables with interval-overlap logic in pandas/numpy and runs scipy.stats.fisher_exact)
- **R** (Alternative fisher.test implementation for odds-ratio computation)
- **matplotlib** (Generates boxplot comparisons of odds ratios across features and label sets (version 3.4.3))

## Examples

```
python predict.py --path ../test --model ../res/model. --output ../res/output.txt; then scipy.stats.fisher_exact(table) per feature to fill enrichment_odds_ratios.csv
```

## Evaluation signals

- enrichment_odds_ratios.csv exists and contains an odds-ratio entry for every regulatory feature x label-set combination
- PE/PP interaction odds ratios are >1 (enrichment) for both true and predicted label sets
- H3K27me3 and H3K9me3 odds ratios are <1 (depletion) as reported for repressive marks
- Predicted-label H3K36me3 median odds ratio approximates 3.71 and true-label approximates 2.62
- Both true and predicted labels show H3K36me3 enrichment across all tissues, with predicted median OR (3.71) exceeding true median OR (2.62)

## Limitations

- Findings are grounded in GTEx cis-eQTL labels across 44 tissues and Roadmap/3DIV annotations; the enrichment pattern may not transfer to other epigenomic datasets or mark sets
- The reported H3K36me3 median ORs (3.71 predicted vs 2.62 true) are approximate targets and depend on the exact interval-overlap definition used
- Fisher's exact test odds ratios quantify overlap but do not by themselves establish causal regulatory impact of the indels

## Evidence

- [abstract] we perform an enrichment analysis for both true and predicted sindels in key regulatory regions such as chromatin interactions, open chromatin regions and histone modification sites: "we perform an enrichment analysis for both true and predicted sindels in key regulatory regions such as chromatin interactions, open chromatin regions and histone modification sites"
- [other] Apply Fisher's exact test (scipy.stats.fisher_exact or R fisher.test) to each contingency table to obtain odds ratios: [evidence span withheld — anchor status: not_found]
- [results] Both true and predicted labels are enriched in H3K36me3 across all tissues, and predicted labels show a much higher enrichment for all tissues, with a median odds ratio of 3.71 for predicted labels versus 2.62 for true labels.: [evidence span withheld — anchor status: not_found]
- [readme] we use the aforementioned steps ``predict.R" and ``predict.py" to generate the input features: "Then, we call ``predict.py" to predict the functional scores."
