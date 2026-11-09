---
name: drug-sensitivity-group-comparison
description: Use when a drug-sensitivity table (e.g. CCLE pharmacological profiles)
  can be split into mutation-defined groups — such as BRAF+H3K27me3/PRC2 double mutants
  versus BRAF-only — and the goal is a two-sample test of response difference.
when_to_use_negative:
- You only have mRNA expression levels of a transcriptional regulator and no per-cell-line
  drug-sensitivity values — use the RePhine association workflow instead of a group
  t-test.
- Groups are defined by a continuous covariate rather than discrete mutation categories
  (e.g. copy-number-adjusted expression) — regression-based association is more appropriate.
- No mutation annotations are available to form the BRAF+PRC2 and BRAF-only groups,
  so the two-sample comparison cannot be constructed.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3301
- http://edamontology.org/topic_3053
tools:
- name: RePhine
  role: Upstream method whose prediction (EZH2/PRC2 loss-of-function reduces PLX4720
    sensitivity) motivates and is validated by this group-comparison test; applied
    to CCLE pharmacogenomic data
  repo: https://github.com/coexps/RePhine
- name: R
  role: Used to load CCLE mutation and drug-sensitivity data frames, group cell lines,
    and run t.test for the two-sample comparison
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Available in the RePhine repository for supporting pharmacogenomic data processing
    (RP score calculation scripts) alongside the R analysis
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:45b10eac542f8eaf59630addef9dea9f4a9c390176165d52e80f415735224035
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - R
  - RePhine
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3301
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# drug-sensitivity-group-comparison

## Summary

Statistically compares pharmacological sensitivity measurements (e.g. CCLE PLX4720 response values) between mutation-defined cell-line groups, testing whether an alteration such as loss-of-function of EZH2/PRC2 changes drug response.

## When to use

Use when a drug-sensitivity table (e.g. CCLE pharmacological profiles) can be split into mutation-defined groups — such as BRAF+H3K27me3/PRC2 double mutants versus BRAF-only — and the goal is a two-sample test of response difference.

## When NOT to use

- You only have mRNA expression levels of a transcriptional regulator and no per-cell-line drug-sensitivity values — use the RePhine association workflow instead of a group t-test.
- Groups are defined by a continuous covariate rather than discrete mutation categories (e.g. copy-number-adjusted expression) — regression-based association is more appropriate.
- No mutation annotations are available to form the BRAF+PRC2 and BRAF-only groups, so the two-sample comparison cannot be constructed.

## Inputs

- CCLE mutation annotation table (cell line x mutation status, including BRAF and H3K27me3/PRC2-related genes such as EZH2)
- CCLE PLX4720 drug-sensitivity/response values per cell line

## Outputs

- Group means, t statistic, and P value from the two-sample t-test (expected P = 0.044)
- Result record written to ttest_plx4720_double_mutant.csv
- Grouping assignment of each cell line as BRAF+H3K27me3 double-mutant or BRAF-only

## How to apply

Load the CCLE mutation annotation table and the PLX4720 drug-sensitivity values into an R data frame, then classify each cell line as double-mutant if it carries both a BRAF mutation and loss-of-function alterations in H3K27me3/PRC2-related genes (e.g. EZH2), otherwise as BRAF-only when only BRAF is mutated. Extract the PLX4720 sensitivity measurements for the two groups and run a two-sample t-test (R t.test) comparing the mean response. Record the group means, t statistic, and P value in a result record and write it to ttest_plx4720_double_mutant.csv. The rationale is the RePhine prediction that loss-of-function of EZH2/PRC2 reduces cancer cell sensitivity toward the BRAF inhibitor PLX4720, so the double-mutant group should show a significantly lower response (reported P = 0.044, Figure S8E).

## Related tools

- **RePhine** (Upstream method whose prediction (EZH2/PRC2 loss-of-function reduces PLX4720 sensitivity) motivates and is validated by this group-comparison test; applied to CCLE pharmacogenomic data) — https://github.com/coexps/RePhine
- **R** (Used to load CCLE mutation and drug-sensitivity data frames, group cell lines, and run t.test for the two-sample comparison) — https://github.com/coexps/RePhine
- **Python** (Available in the RePhine repository for supporting pharmacogenomic data processing (RP score calculation scripts) alongside the R analysis) — https://github.com/coexps/RePhine

## Examples

```
tt <- t.test(plx4720$Response[grp=='double_mutant'], plx4720$Response[grp=='braf_only']); write.csv(data.frame(mean1=tt$estimate[1], mean2=tt$estimate[2], t=tt$statistic, p=tt$p.value), 'ttest_plx4720_double_mutant.csv')
```

## Evaluation signals

- Computed two-sample t-test P value should match the reported P = 0.044 for the PLX4720 double-mutant versus BRAF-only comparison (Figure S8E).
- The double-mutant (BRAF + H3K27me3/PRC2 loss-of-function) group mean PLX4720 response must be significantly lower than the BRAF-only group mean.
- Every cell line receives exactly one group label (double-mutant vs BRAF-only), and the group membership counts should cover all BRAF-mutant lines in the CCLE table.
- Output file ttest_plx4720_double_mutant.csv contains the record with group means, t statistic, and P value as required by the workflow.

## Limitations

- Detection of acquired resistance is limited due to lack of post-treatment data in CCLE; the comparison reflects pre-treatment associations only.
- The relationship may not be causal — upstream regulators or kinases may influence drug response and simultaneously regulate downstream TRs; TR knockout/activation experiments combined with drug response examination are required to validate causality.
- Cell lines lacking PLX4720 sensitivity measurements or ambiguous H3K27me3/PRC2 loss-of-function annotations cannot be cleanly assigned to either group, reducing power.

## Evidence

- [abstract] RePhine predicted that loss-of-function of EZH2/PRC2 reduces cancer cell sensitivity toward the BRAF inhibitor PLX4720.: "RePhine predicted that loss-of-function of EZH2/PRC2 reduces cancer cell sensitivity toward the BRAF inhibitor PLX4720. Experimental validation confirmed that pharmacological EZH2 inhibition"
- [discussion] Causal validation requirement for TR-drug response findings: "TR target analyses and TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship"
- [readme] RePhine method description from repository README: "RePhine is a method for identification of drug response related transcriptional regulators by modeling correlation patterns of targets."
- [readme] Mutation-based drug-response association step in RePhine workflow: "Identify mutated genes related to drug response based on the CCLE mutation data and pharmacology profiles through an adaptive lasso model and likelihood ratio test."
