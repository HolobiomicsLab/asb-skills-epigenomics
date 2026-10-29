---
name: enrichment-odds-ratio-visualization
description: Use when a set of labeled regulatory variants (true cis-eQTL indels vs
  model-predicted scores) must be tested for enrichment/depletion in interval annotations
  (promoter-centered chromatin interactions, open chromatin, histone marks) across
  multiple tissues, with odds-ratio comparison tables and.
when_to_use_negative:
- The input is already a computed odds-ratio or contingency-table summary — only the
  visualization step applies, not the overlap testing.
- The query variants are SNVs or other variant classes rather than noncoding small
  indels (sindels), which is the scope of TIVAN-indel labels and precomputed scores.
- Regulatory annotations are not interval-based (e.g. point-mutation signature data),
  since the method relies on interval-overlap contingency tables.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: TIVAN-indel
  role: Supplies the predicted nc-sindel functional scores whose enrichment pattern
    is compared against true cis-eQTL labels
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: pandas
  role: Interval-overlap logic to build 2x2 contingency tables of sindel overlap vs
    regulatory features
- name: numpy
  role: Array operations supporting the overlap/contingency-table construction
- name: matplotlib
  role: Boxplot comparison of odds ratios across feature/label-set combinations (pinned
    to 3.4.3)
- name: R
  role: Alternative Fisher's exact test implementation via fisher.test (R 4.1.1)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad060
    title: 'TIVAN-indel: a computational framework for annotating and predicting non-coding
      regulatory small insertions and deletions'
  source_content_hash: sha256:f4601a8b81896132d435dd761bd70a0efe2e6f686ae3e285a81f8edc5b2f864d
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# enrichment-odds-ratio-visualization

## Summary

Reproduces the TIVAN-indel paper's enrichment analysis by testing overlap of true vs predicted nc-sindels with 3DIV PE/PP chromatin interactions, open-chromatin and seven Roadmap histone marks via Fisher's exact test, then comparing median odds ratios (e.g. H3K36me3: 3.71 predicted vs 2.62 true) and visualizing them as boxplots.

## When to use

Use when a set of labeled regulatory variants (true cis-eQTL indels vs model-predicted scores) must be tested for enrichment/depletion in interval annotations (promoter-centered chromatin interactions, open chromatin, histone marks) across multiple tissues, with odds-ratio comparison tables and plots as deliverables.

## When NOT to use

- The input is already a computed odds-ratio or contingency-table summary — only the visualization step applies, not the overlap testing.
- The query variants are SNVs or other variant classes rather than noncoding small indels (sindels), which is the scope of TIVAN-indel labels and precomputed scores.
- Regulatory annotations are not interval-based (e.g. point-mutation signature data), since the method relies on interval-overlap contingency tables.

## Inputs

- True nc-sindel label set from GTEx cis-eQTL analyses across 44 tissues
- Predicted nc-sindel scores from TIVAN-indel (e.g. Adipose_Subcutaneous.score.rda precomputed tissue-specific score files)
- 3DIV PE/PP promoter-centered chromatin interaction interval annotations
- Roadmap open-chromatin interval annotations
- Roadmap annotations for seven histone marks (including H3K36me3, H3K27me3, H3K9me3)

## Outputs

- enrichment_odds_ratios.csv with one odds-ratio entry per regulatory feature x label-set combination
- Boxplot comparisons of odds ratios across label sets and tissues (Figures 7-8 style, matplotlib 3.4.3)
- Per-tissue Fisher's exact test odds ratios for each regulatory feature

## How to apply

Load the true nc-sindel labels (GTEx cis-eQTL derived) and predicted labels (TIVAN-indel scores from 'python predict.py --path ../test --model ../res/model. --output ../res/output.txt') alongside 3DIV PE/PP promoter-centered interaction intervals and Roadmap open-chromatin plus seven histone-mark interval annotations. For each regulatory feature x label set x tissue, build 2x2 contingency tables of sindel overlap vs non-overlap using interval-overlap logic in pandas/numpy, then apply Fisher's exact test (scipy.stats.fisher_exact or R fisher.test) to get odds ratios. Summarize median odds ratios per feature and label set into enrichment_odds_ratios.csv and render cross-label-set boxplot comparisons with matplotlib 3.4.3 (Figures 7-8 style). Validate against the paper's reported pattern: PE/PP OR>1, repressive marks H3K27me3 and H3K9me3 OR<1, and predicted-label H3K36me3 median OR of 3.71 versus 2.62 for true labels.

## Related tools

- **TIVAN-indel** (Supplies the predicted nc-sindel functional scores whose enrichment pattern is compared against true cis-eQTL labels) — https://github.com/lichen-lab/TIVAN-indel
- **pandas** (Interval-overlap logic to build 2x2 contingency tables of sindel overlap vs regulatory features)
- **numpy** (Array operations supporting the overlap/contingency-table construction)
- **matplotlib** (Boxplot comparison of odds ratios across feature/label-set combinations (pinned to 3.4.3))
- **R** (Alternative Fisher's exact test implementation via fisher.test (R 4.1.1))

## Examples

```
from scipy.stats import fisher_exact; or_, p = fisher_exact([[overlap_pos, nonoverlap_pos], [overlap_neg, nonoverlap_neg]])
```

## Evaluation signals

- enrichment_odds_ratios.csv exists and contains an odds-ratio entry for every regulatory feature x label-set combination.
- PE/PP interaction enrichment shows OR > 1 for both true and predicted labels.
- Repressive histone marks H3K27me3 and H3K9me3 show OR < 1 (depletion).
- H3K36me3 median odds ratio for predicted labels approximates 3.71, versus 2.62 for true labels, with predicted enrichment higher in all tissues.
- Boxplots render with separate distributions for true vs predicted label sets across tissues.

## Limitations

- Enrichment patterns are validated only against 3DIV pcHi-C interactions and Roadmap histone/open-chromatin annotations; other regulatory resources are not covered.
- Predicted labels show higher enrichment than true labels (median OR 3.71 vs 2.62), so predicted-label ORs cannot be treated as calibrated substitutes for true eQTL enrichment.
- Comparisons rely on median odds ratios across 44 GTEx tissues, masking per-tissue variability; environment pins (R 4.1.1, matplotlib 3.4.3, pandas 1.4.0, numpy 1.19.3) are needed for reproducibility.

## Evidence

- [abstract] we perform an enrichment analysis for both true and predicted sindels in key regulatory regions such as chromatin interactions, open chromatin regions and histone modification sites: "we perform an enrichment analysis for both true and predicted sindels in key regulatory regions such as chromatin interactions, open chromatin regions and histone modification sites"
- [other] Apply Fisher's exact test (scipy.stats.fisher_exact or R fisher.test) to each contingency table to obtain odds ratios quantifying enrichment or depletion of true and predicted sindels in each regulatory feature.: [evidence span withheld — anchor status: not_found]
- [results] Both true and predicted labels are enriched in H3K36me3 across all tissues, and predicted labels show a much higher enrichment for all tissues, with a median odds ratio of 3.71 for predicted labels versus 2.62 for true labels.: [evidence span withheld — anchor status: not_found]
- [readme] TIVAN-Indel is implemented by both R and Python and the requirements are: "TIVAN-Indel is implemented by both R and Python and the requirements are"
