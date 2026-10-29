---
name: chromatin-interaction-sindel-annotation
description: Use when validating that predicted noncoding indels are biologically
  meaningful by testing their enrichment in enhancer/promoter chromatin interactions
  or histone-mark regions against true cis-eQTL sindels.
when_to_use_negative:
- The goal is to train or score the TIVAN-indel model itself rather than validate
  its predictions biologically — use train.py/predict.py workflows instead
- Input is SNP/variant data rather than small indels (sindels); TIVAN-indel and this
  enrichment analysis are specific to noncoding small insertions and deletions
- Regulatory annotations are not interval-based (e.g., only quantitative per-base
  signals without genomic coordinates), making 2x2 overlap contingency tables ill-defined
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: TIVAN-indel
  role: Supervised framework providing the predicted noncoding-sindel functional scores
    whose enrichment pattern is compared to true cis-eQTL sindels
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: Python
  role: Implementation language for interval-overlap contingency tables, scipy Fisher's
    exact test, and matplotlib boxplots
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: pandas
  role: Loading label sets and interval annotations and computing sindel overlap vs
    non-overlap
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: numpy
  role: Array-based interval-overlap logic for building 2x2 contingency tables
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: matplotlib
  role: Generating boxplot comparisons of odds ratios (Figures 7-8 style), version
    3.4.3
  repo: https://github.com/lichen-lab/TIVAN-indel
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad060
    title: 'TIVAN-indel: a computational framework for annotating and predicting non-coding
      regulatory small insertions and deletions'
  source_content_hash: sha256:a6c2cf390935cf7a8c3af74f73876176247c8fdb265049572779a427b02f4ed6
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
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# chromatin-interaction-sindel-annotation

## Summary

Enrichment analysis of true and TIVAN-indel-predicted noncoding sindels in regulatory regions (3DIV PE/PP chromatin interactions, Roadmap open chromatin, and seven histone marks) via 2x2 contingency tables and Fisher's exact test, to verify that predicted sindels reproduce the enrichment pattern of true regulatory sindels.

## When to use

Use when validating that predicted noncoding indels are biologically meaningful by testing their enrichment in enhancer/promoter chromatin interactions or histone-mark regions against true cis-eQTL sindels.

## When NOT to use

- The goal is to train or score the TIVAN-indel model itself rather than validate its predictions biologically — use train.py/predict.py workflows instead
- Input is SNP/variant data rather than small indels (sindels); TIVAN-indel and this enrichment analysis are specific to noncoding small insertions and deletions
- Regulatory annotations are not interval-based (e.g., only quantitative per-base signals without genomic coordinates), making 2x2 overlap contingency tables ill-defined

## Inputs

- True noncoding-sindel label sets from GTEx cis-eQTL analyses across 44 tissues
- Predicted sindel functional scores from TIVAN-indel (e.g., ../res/output.txt)
- 3DIV PE/PP promoter-centered chromatin interaction intervals
- Roadmap open-chromatin interval annotations
- Roadmap seven histone-mark interval annotations (including H3K36me3, H3K27me3, H3K9me3)

## Outputs

- enrichment_odds_ratios.csv with a Fisher's exact test odds ratio for every regulatory feature x label-set combination
- Boxplot comparisons of odds ratios per feature and label set (Figures 7-8 style) generated with matplotlib

## How to apply

Load true noncoding-sindel labels from GTEx cis-eQTL analyses and predicted functional scores from TIVAN-indel (predicted via 'python predict.py'), together with 3DIV PE/PP promoter-centered interaction intervals and Roadmap open-chromatin and seven histone-mark interval annotations. For each regulatory feature (PE/PP interactions, open chromatin, H3K36me3, H3K27me3, H3K9me3, etc.) and each label set, build 2x2 contingency tables of sindel overlap vs non-overlap using interval-overlap logic in pandas/numpy. Apply Fisher's exact test (scipy.stats.fisher_exact or R fisher.test) to obtain odds ratios quantifying enrichment or depletion. Compare odds ratios across label sets: expect PE/PP OR>1, depletion (OR<1) for repressive marks H3K27me3 and H3K9me3, and a higher H3K36me3 median OR for predicted labels (3.71) than true labels (2.62). Summarize median odds ratios per feature and label set in enrichment_odds_ratios.csv and generate boxplot comparisons with matplotlib==3.4.3 (Figures 7-8 style).

## Related tools

- **TIVAN-indel** (Supervised framework providing the predicted noncoding-sindel functional scores whose enrichment pattern is compared to true cis-eQTL sindels) — https://github.com/lichen-lab/TIVAN-indel
- **Python** (Implementation language for interval-overlap contingency tables, scipy Fisher's exact test, and matplotlib boxplots) — https://github.com/lichen-lab/TIVAN-indel
- **pandas** (Loading label sets and interval annotations and computing sindel overlap vs non-overlap) — https://github.com/lichen-lab/TIVAN-indel
- **numpy** (Array-based interval-overlap logic for building 2x2 contingency tables) — https://github.com/lichen-lab/TIVAN-indel
- **matplotlib** (Generating boxplot comparisons of odds ratios (Figures 7-8 style), version 3.4.3) — https://github.com/lichen-lab/TIVAN-indel

## Examples

```
python predict.py --path ../test --model ../res/model. --output ../res/output.txt; then test overlap tables with scipy.stats.fisher_exact for each regulatory feature x label set
```

## Evaluation signals

- enrichment_odds_ratios.csv exists and contains an odds-ratio entry for every regulatory feature x label-set combination
- PE/PP chromatin-interaction odds ratio is >1 (enrichment) for both true and predicted label sets
- H3K27me3 and H3K9me3 odds ratios are <1 (depletion) for repressive marks
- Predicted-label H3K36me3 median odds ratio approximates 3.71 and true-label approximates 2.62, with predicted > true
- Boxplots reproduce the Figures 7-8 comparison of odds-ratio distributions across label sets

## Limitations

- The 3DIV/roadmap enrichment patterns are tissue-specific; median odds ratios (3.71 predicted vs 2.62 true for H3K36me3) apply to the tissues analyzed and may not transfer
- Higher predicted-label enrichment than true labels (e.g., OR 3.71 vs 2.62 for H3K36me3) indicates the model favors annotated regulatory regions and does not by itself establish causal regulation
- Interval-overlap contingency tables depend on the interval definition of regulatory features; no changelog/discussion details were available for the source document

## Evidence

- [abstract] we perform an enrichment analysis for both true and predicted sindels in key regulatory regions such as chromatin interactions, open chromatin regions and histone modification sites: "we perform an enrichment analysis for both true and predicted sindels in key regulatory regions such as chromatin interactions, open chromatin regions and histone modification sites"
- [results] predicted labels show a much higher enrichment for all tissues, with a median odds ratio of 3.71 for predicted labels versus 2.62 for true labels: [evidence span withheld — anchor status: not_found]
- [intro] leverage labelled noncoding sindels identified by cis-expression quantitative trait loci (cis- eQTLs) analyses across 44 tissues in GTEx: "By leveraging labelled noncoding sindels identified by cis-expression quantitative trait loci (cis- eQTLs) analyses across 44 tissues in GTEx, and a compilation of both generic functional annotations"
- [readme] python predict.py --path ../test --model ../res/model. --output ../res/output.txt: [evidence span withheld — anchor status: not_found]
