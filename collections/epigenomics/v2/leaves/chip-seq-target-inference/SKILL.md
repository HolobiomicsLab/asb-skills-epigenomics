---
name: chip-seq-target-inference
description: Use when you need to infer which genes a transcriptional regulator targets
  from ChIP-seq peak data (e.g., ENCODE), especially as input to regression-based
  drug response association such as RePhine on CCLE data.
when_to_use_negative:
- No ChIP-seq data exist for the TR of interest, or the available ChIP-seq data have
  too few targets — the article states this 'would mislead the identification of TRs'
- Reliable target information is unavailable for the TR — RePhine is restricted to
  cases where reliable target information exists
- The goal is to detect acquired (post-treatment) resistance mechanisms, e.g., acquired
  STAT3 activation for erlotinib, which RePhine could not detect from CCLE pre-treatment
  data
edam_operation: http://edamontology.org/operation_2575
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3577
tools:
- name: RePhine
  role: R package that consumes RP scores from ChIP-seq target inference and performs
    regression to identify TRs associated with drug response
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Runs RP_calculation_modified.py, the modified script that calculates RP scores
    considering both peak distance and strength from ChIP-seq peak BED files
  repo: https://github.com/coexps/RePhine
- name: R
  role: Hosts the RePhine package (Rephine_1.1.0) and drug_code_R.R used for the regression,
    likelihood ratio tests, elastic net, and GSEA filtering
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:e4e14673528b07330254857590a59e249563b71e322ea7a8903bd75a8f4e9317
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - R
  - RePhine
  edam_operation: http://edamontology.org/operation_2575
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ChIP-seq target inference for transcriptional regulator association

## Summary

Quantitative inference of transcriptional regulator (TR) targets from ENCODE ChIP-seq data by combining peak signal strength and the distances from peaks to targets, producing regulatory potential (RP) scores that RePhine uses to link TRs to drug response in CCLE pharmacogenomic data.

## When to use

Use when you need to infer which genes a transcriptional regulator targets from ChIP-seq peak data (e.g., ENCODE), especially as input to regression-based drug response association such as RePhine on CCLE data.

## When NOT to use

- No ChIP-seq data exist for the TR of interest, or the available ChIP-seq data have too few targets — the article states this 'would mislead the identification of TRs'
- Reliable target information is unavailable for the TR — RePhine is restricted to cases where reliable target information exists
- The goal is to detect acquired (post-treatment) resistance mechanisms, e.g., acquired STAT3 activation for erlotinib, which RePhine could not detect from CCLE pre-treatment data

## Inputs

- ENCODE ChIP-seq peak BED files per transcriptional regulator
- CCLE gene expression matrix
- CCLE drug sensitivity (pharmacological profile) data, e.g., PLX4720
- CCLE copy number and mutation status data for confounder adjustment
- Optionally, precomputed regulatory potential (RP) scores from the RePhine-distributed Google Drive resource

## Outputs

- Regulatory potential (RP) scores per gene for each ChIP-seq TR (computed by RP_calculation_modified.py)
- TR ranking table with RePhine coefficient, uniP, multiP, and permuP statistics per drug (e.g., tr_ranking_plx4720.csv)
- Sets of positively- and negatively-correlated independent TRs passing uniP < 1E-5 and multiP < 0.005

## How to apply

Obtain ENCODE ChIP-seq peak BED files and run the modified Python script RP_calculation_modified.py, which computes regulation potential (RP) scores by considering both ChIP-seq signal strength and the distances from peaks to targets. Precomputed RP scores across more than 1000 ENCODE ChIP-seq datasets are also distributed via the RePhine Google Drive. Use these target-level scores in the RePhine regression, which computes partial correlations between genes and a drug (e.g., PLX4720 response in CCLE) while controlling confounders (CNVs, kinase/tumor-suppressor mutations), then evaluates TRs whose targets show concordant partial correlations via likelihood ratio tests (uniP) and elastic net combined with likelihood ratio tests (multiP). Because RePhine identifies TRs through their targets, accurate quantitative target inference is critical; low-quality or sparse ChIP-seq data will propagate errors into TR identification.

## Related tools

- **RePhine** (R package that consumes RP scores from ChIP-seq target inference and performs regression to identify TRs associated with drug response) — https://github.com/coexps/RePhine
- **Python** (Runs RP_calculation_modified.py, the modified script that calculates RP scores considering both peak distance and strength from ChIP-seq peak BED files) — https://github.com/coexps/RePhine
- **R** (Hosts the RePhine package (Rephine_1.1.0) and drug_code_R.R used for the regression, likelihood ratio tests, elastic net, and GSEA filtering) — https://github.com/coexps/RePhine

## Examples

```
python RP_calculation_modified.py --input TF_peaks.bed
```

## Evaluation signals

- Output TR ranking recovers GTF3C2, YY1, ESR1, E2F1, MYC, GATA3, RBBP5, EZH2, E2F4, ZEB1, ZNF217 as negatively-correlated TRs and SPI1, CEBPB, EP300 as positively-correlated TRs for PLX4720 at uniP < 1e-5 and multiP < 0.005 (Figure 4A / Table S8)
- RP score calculation consumes ChIP-seq peak BED files and incorporates both peak strength and peak-to-target distance, not signal alone
- Permutation test significance (permuP) is computed and reported alongside uniP and multiP for each TR
- Downstream clustering with RePhine-derived TR signatures separates drugs with different mechanisms of action on CCLE data
- Known positive controls behave as expected, e.g., EZH2 inhibition increasing resistance to PLX4720 and ZEB1 resistance reflected at the mRNA level

## Limitations

- Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs
- RePhine identifies drug response-related TRs through targets, so it is restricted to cases where reliable target information is available
- Relationships inferred may not be causal; upstream regulators or kinases may influence drug response while regulating downstream TRs
- Detection of acquired resistance is limited due to lack of post-treatment data in CCLE (e.g., secondary resistance to erlotinib via acquired STAT3 activation could not be detected)

## Evidence

- [discussion] RePhine task 3: quantitative target inference using ChIP-seq signals and peak-to-target distances: "application of quantitative and informative target inference by considering both ChIP-seq signals and the distances from peaks to the targets"
- [readme] RP script described in README: [evidence span withheld — anchor status: external_doc]
- [readme] Method steps in README: "Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response"
- [results] PLX4720 validation threshold: [evidence span withheld — anchor status: not_found]
- [discussion] Restriction to reliable targets: "because RePhine identifies drug response-related TRs through the targets, it is restricted to the cases where reliable target information is available"
