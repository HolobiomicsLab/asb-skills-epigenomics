---
name: mutation-status-cell-line-partitioning
description: Use when CCLE mutation annotation and drug-sensitivity (e.g. PLX4720)
  tables are available and the goal is to test whether a co-mutation class (BRAF +
  PRC2/H3K27me3 loss) shifts pharmacological response versus a single-mutant baseline.
when_to_use_negative:
- 'No post-treatment data are available: RePhine''s CCLE analysis lacks post-treatment
  data, so acquired-resistance mechanisms (e.g. acquired STAT3 activation driving
  erlotinib resistance) cannot be detected from pre-treatment mutation/drug-response
  associations'
- The cell lines being partitioned do not carry BRAF mutations, since the comparison
  is anchored on BRAF-mutant lines
- Only mRNA levels are available without mutation/pharmacology profiles — mRNA-only
  TR–drug prediction is described as not sufficiently robust due to inherent experimental
  noises
edam_operation: http://edamontology.org/operation_2239
tools:
- name: R
  role: Used to load CCLE mutation and pharmacology tables, partition cell lines by
    mutation class, and run the two-sample t-test (t.test)
  repo: https://github.com/coexps/RePhine
- name: RePhine
  role: Upstream method that predicted the EZH2/PRC2 loss-of-function → reduced PLX4720
    sensitivity association being validated by the t-test comparison
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Used alongside R for the CCLE data processing (modified Python scripts of
    RP score calculation are provided in the repo)
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:8f13d182d6a5edc32d19018ce6bac7b7d4fffd46248226201626d171d4af286b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - R
  - RePhine
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mutation-status-cell-line-partitioning

## Summary

Stratifies CCLE cancer cell lines by mutation status — BRAF-mutant lines split into double-mutants (BRAF plus loss-of-function alterations in H3K27me3/PRC2 genes such as EZH2) versus BRAF-only — and tests whether the two groups differ in PLX4720 drug-sensitivity response using a two-sample t-test.

## When to use

Use when CCLE mutation annotation and drug-sensitivity (e.g. PLX4720) tables are available and the goal is to test whether a co-mutation class (BRAF + PRC2/H3K27me3 loss) shifts pharmacological response versus a single-mutant baseline.

## When NOT to use

- No post-treatment data are available: RePhine's CCLE analysis lacks post-treatment data, so acquired-resistance mechanisms (e.g. acquired STAT3 activation driving erlotinib resistance) cannot be detected from pre-treatment mutation/drug-response associations
- The cell lines being partitioned do not carry BRAF mutations, since the comparison is anchored on BRAF-mutant lines
- Only mRNA levels are available without mutation/pharmacology profiles — mRNA-only TR–drug prediction is described as not sufficiently robust due to inherent experimental noises

## Inputs

- CCLE mutation annotation table (mutation status per cell line, including BRAF and H3K27me3/PRC2-related genes such as EZH2)
- CCLE PLX4720 drug-sensitivity values (pharmacological profiles per cell line)

## Outputs

- Cell-line partition into double-mutant (BRAF + H3K27me3/PRC2 loss-of-function) and BRAF-only groups
- Two-sample t-test result record with group means, t statistic, and P value written to ttest_plx4720_double_mutant.csv

## How to apply

Load the CCLE mutation annotation table and PLX4720 drug-sensitivity values into an R data frame, then classify each cell line as double-mutant if it carries both a BRAF mutation and loss-of-function alterations in H3K27me3/PRC2-related genes (e.g. EZH2), or BRAF-only when only BRAF is mutated. Extract the PLX4720 sensitivity measurements for the two groups and run a two-sample t-test (t.test in R) comparing mean response. Record group means, the t statistic, and the P value in a result record written to ttest_plx4720_double_mutant.csv. Rationale: the RePhine analysis predicted that loss-of-function of EZH2/PRC2 reduces cancer cell sensitivity toward the BRAF inhibitor PLX4720, so the double-mutant group should show a significantly lower response.

## Related tools

- **R** (Used to load CCLE mutation and pharmacology tables, partition cell lines by mutation class, and run the two-sample t-test (t.test)) — https://github.com/coexps/RePhine
- **RePhine** (Upstream method that predicted the EZH2/PRC2 loss-of-function → reduced PLX4720 sensitivity association being validated by the t-test comparison) — https://github.com/coexps/RePhine
- **Python** (Used alongside R for the CCLE data processing (modified Python scripts of RP score calculation are provided in the repo)) — https://github.com/coexps/RePhine

## Examples

```
t.test(plx4720_response[cell_line_class == "double_mutant"], plx4720_response[cell_line_class == "BRAF_only"], var.equal = TRUE)
```

## Evaluation signals

- The computed two-sample t-test P value should match the reported P = 0.044
- The double-mutant (BRAF + H3K27me3/PRC2) group mean PLX4720 response must be significantly LOWER than the BRAF-only group mean
- Every cell line in the partition carries a BRAF mutation; BRAF-wild-type lines are excluded from the comparison
- The result record ttest_plx4720_double_mutant.csv must contain group means, t statistic, and P value
- The direction of the effect should be consistent with RePhine's prediction that pharmacological EZH2 inhibition increases resistance to PLX4720

## Limitations

- The finding is an association, not causal: TR target analyses and TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship
- Confounding mutations of kinase genes or tumor suppressor genes, CNVs, and data noises can affect mutation-status stratification
- Detection of acquired resistance is limited due to lack of post-treatment data in CCLE

## Evidence

- [abstract] RePhine predicted that loss-of-function of EZH2/PRC2 reduces cancer cell sensitivity toward PLX4720, and pharmacological EZH2 inhibition increased resistance to PLX4720: "RePhine predicted that loss-of-function of EZH2/PRC2 reduces cancer cell sensitivity toward the BRAF inhibitor PLX4720."
- [supplementary] workflow t-test step: [evidence span withheld — anchor status: not_found]
- [supplementary] validation expectation: [evidence span withheld — anchor status: not_found]
- [discussion] causality caveat: "TR target analyses and TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship"
- [readme] README prerequisites: [evidence span withheld — anchor status: external_doc]
