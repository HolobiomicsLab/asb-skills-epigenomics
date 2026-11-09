---
name: multiple-testing-fdr-correction
description: Use when a CITRUS in silico knock out or association analysis yields
  one raw p-value per TF (hundreds of TFs tested simultaneously) and significant TF–alteration
  or TF-overlap associations must be controlled for multiple testing.
when_to_use_negative:
- When only a single hypothesis is tested (e.g., one TF against one alteration) —
  no correction is needed and BH adjustment would be conservative.
- When the goal is ranking or effect-size estimation of TF activities rather than
  calling significance; FDR q-values alter ranking and should not replace effect sizes.
- When p-values are not independent or positively correlated in a BH-compatible manner
  without verification — e.g., heavily correlated TF families sharing motifs.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: SciPy
  role: Provides the two-sample t-tests producing raw per-TF p-values; can be used
    for multiple-testing utilities
- name: statsmodels
  role: Performs Benjamini-Hochberg FDR correction via multipletests on the per-TF
    p-value vector
- name: Python 3.7
  role: Runtime environment (3.7+) in which the correction and downstream CSV compilation
    scripts execute
  repo: https://github.com/osmanbeyoglulab/CITRUS
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:b047f9719a1efc49b6c77b206d49d466b6736bfdf2ef3d1c8b041a901ad7afc0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/osmanbeyoglulab/CITRUS
  tools:
  - CITRUS
  - Numpy
  - Pandas
  - Python
  - Python 3.7
  - SciPy
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multiple-testing-fdr-correction

## Summary

Applies Benjamini-Hochberg false discovery rate correction to p-value vectors arising from per-TF association testing (e.g., two-sample t-tests of CITRUS-inferred TF activities against somatic alteration status), so that significant TF–alteration associations are reported at FDR-adjusted P-value < 0.05.

## When to use

Use when a CITRUS in silico knock out or association analysis yields one raw p-value per TF (hundreds of TFs tested simultaneously) and significant TF–alteration or TF-overlap associations must be controlled for multiple testing.

## When NOT to use

- When only a single hypothesis is tested (e.g., one TF against one alteration) — no correction is needed and BH adjustment would be conservative.
- When the goal is ranking or effect-size estimation of TF activities rather than calling significance; FDR q-values alter ranking and should not replace effect sizes.
- When p-values are not independent or positively correlated in a BH-compatible manner without verification — e.g., heavily correlated TF families sharing motifs.

## Inputs

- Vector of raw per-TF p-values from SciPy two-sample t-tests comparing mutant vs wild-type TF activity distributions (from 'hid_tmr' in output_dataset_CITRUS.pkl)
- Per-TF p-values from association tests of TF activity with PIK3CA mutation status across 5803 TCGA samples
- FDR significance threshold alpha = 0.05

## Outputs

- FDR-adjusted q-values per TF (Benjamini-Hochberg)
- tf_alteration_associations.csv listing significant TF–alteration pairs (FDR < 0.05) per alteration gene and cancer type
- overlap_statistics.csv with Fisher's exact test enrichment statistics for phosphoproteomic (16/20) and AKT-substrate (6/7) TF list overlaps

## How to apply

After computing per-TF raw p-values (e.g., SciPy two-sample t-tests comparing mutant-carrier versus non-carrier TF activity distributions from the 'hid_tmr' ensemble layer of output_dataset_CITRUS.pkl), assemble the p-values into a single vector across all TFs for a given alteration gene and cancer type. Apply Benjamini-Hochberg FDR correction with statsmodels.stats.multitest.multipletests(method='fdr_bh') or the equivalent SciPy multiple-testing utility, using alpha = 0.05 as the significance threshold. Retain TFs whose adjusted p-value (q-value) is < 0.05 and compile these as tf_alteration_associations.csv per alteration gene (PIK3CA, PTEN, KRAS, TP53, CTNNB1 in UCEC; PIK3CA, TP53, GATA3, CDH1 in BRCA). The rationale is that CITRUS scores ~hundreds of TF activities per tumor, so uncorrected t-test p-values would produce an unacceptably high false-positive rate; the same correction is applied before quantifying overlaps with external PIK3CA knock-in phosphoproteomic and AKT1 kinase substrate TF lists via Fisher's exact test.

## Related tools

- **SciPy** (Provides the two-sample t-tests producing raw per-TF p-values; can be used for multiple-testing utilities)
- **statsmodels** (Performs Benjamini-Hochberg FDR correction via multipletests on the per-TF p-value vector)
- **Python 3.7** (Runtime environment (3.7+) in which the correction and downstream CSV compilation scripts execute) — https://github.com/osmanbeyoglulab/CITRUS

## Examples

```
from statsmodels.stats.multitest import multipletests; rej, qvals, _, _ = multipletests(pvals_per_tf, alpha=0.05, method='fdr_bh')
```

## Evaluation signals

- Adjusted q-values must be monotonically non-decreasing when raw p-values are sorted ascending (Benjamini-Hochberg invariant).
- Number of TFs retained at FDR < 0.05 must reproduce the reported associations: PTEN-linked TFs including E2F5, TP63, ELF3, DBP, ZKSCAN3, LHX2, HOXB6, SOX9, MYBL1, GLIS1 in UCEC; CTNNB1-linked TCF7, TCF7L2, TCF7L1, FOXH1, EMX1, MYBL1.
- Overlap counts against external lists must equal 16 of 20 MCF10A PIK3CA knock-in phosphoproteomic TFs and 6 of 7 AKT1-substrate TFs at FDR < 0.05.
- tf_alteration_associations.csv must match the significant TF–alteration pairs shown in Figure 5A-B for UCEC (PIK3CA, PTEN, KRAS, TP53, CTNNB1) and BRCA (PIK3CA, TP53, GATA3, CDH1).
- Schema check: every corrected entry must carry both a raw p-value and an adjusted q-value; no q-value may be smaller than its raw p-value.

## Limitations

- TFs of the same family often share a similar motif and thus are difficult to disambiguate, so corrected associations may reflect family-level motif activity rather than single-TF activity.
- No directionality (activation vs repression) is represented in TF–target gene priors, so a significant FDR-passing association does not indicate sign of regulatory effect.
- Tumor regulatory network analysis is confounded by stromal/immune cells and cancer cell heterogeneity, which can inflate or deflate per-TF p-values before correction.

## Evidence

- [other] Correct the resulting p-values for multiple testing across TFs using FDR correction (Benjamini-Hochberg) via statsmodels or SciPy.: [evidence span withheld — anchor status: not_found]
- [other] FDR threshold in external validation workflow: [evidence span withheld — anchor status: not_found]
- [readme] TF activity layer tested per TF: ""hid_tmr": hid_tmr,       # TF activity"
- [other] Validation threshold for reproducing reported associations: [evidence span withheld — anchor status: not_found]
