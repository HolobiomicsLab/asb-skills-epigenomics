---
name: chip-seq-binding-library-based-tr-identification
description: Use when a set of differentially accessible region BED files (e.g., from
  bulk TR perturbation cKO experiments via DiffBind export.
when_to_use_negative:
- The query input is a gene expression list rather than a genomic region set — BART's
  geneset mode or differential-expression-based TR inference is the appropriate route
  instead of region-based TR ranking.
- No context-specific TR ChIP-seq reference library exists for the organism or genome
  build of the query regions (BIT relies on 10,140 human / 5,681 mouse ChIP-seq datasets),
  forcing motif-only methods.
- The input peak set is already ranked TR output from another method — this skill
  consumes raw coordinate BED regions, not TR ranking tables.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0204
tools:
- name: BIT
  role: 'Primary Bayesian method: transforms query peak coordinates to fixed-length
    binary vectors, fits a Bayesian hierarchical model via Gibbs-sampler MCMC, and
    assigns each TR a BIT_score used for ranking'
  repo: https://github.com/ZeyuL01/BIT
- name: BART
  role: ChIP-seq binding-based comparison method run in region mode on the same scored
    BED input
  repo: https://github.com/zanglab/bart2
- name: ChIP-Atlas
  role: ChIP-seq-based comparison method that ranks TRs using p values from Fisher's
    exact tests
- name: I-cisTarget
  role: Motif-based comparison method using ENCODE references and recovery analysis
- name: WhichTF
  role: Motif-based comparison method using JASPAR/TRANSFAC references and hypergeometric
    tests
- name: HOMER
  role: Motif-based comparison method leveraging TR binding motifs as reference
- name: R
  role: Language of the BIT R package (GitHub) used to run the method and compute
    MRR; R DiffBind used to generate DAR BED inputs
  repo: https://github.com/ZeyuL01/BIT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:07f63ae478e492861a037c600218176f5a69cf61904c2e17180190424f596812
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZeyuL01/BIT
  repo_url: https://github.com/ZeyuL01/BIT
  tools:
  - BART
  - BIT
  - ChIP-Atlas
  - HOMER
  - I-cisTarget
  - R
  - WhichTF
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# chip-seq-binding-library-based-tr-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify and rank transcriptional regulators (TRs) from user-provided epigenomic query region sets (e.g., ATAC-seq DAR BED files) by assessing genome-wide consistency against large TR ChIP-seq binding libraries (BIT: 10,140 human / 5,681 mouse TR ChIP-seq datasets), and benchmark the resulting TR rankings against BART, ChIP-Atlas, i-cisTarget, WhichTF, and HOMER using mean reciprocal rank (MRR) on perturbed TRs.

## When to use

Use when a set of differentially accessible region BED files (e.g., from bulk TR perturbation cKO experiments via DiffBind export.bed) must be converted into a ranked TR list, and the rankings of ChIP-seq-library-based and motif-based TR-identification methods must be quantitatively compared via MRR.

## When NOT to use

- The query input is a gene expression list rather than a genomic region set — BART's geneset mode or differential-expression-based TR inference is the appropriate route instead of region-based TR ranking.
- No context-specific TR ChIP-seq reference library exists for the organism or genome build of the query regions (BIT relies on 10,140 human / 5,681 mouse ChIP-seq datasets), forcing motif-only methods.
- The input peak set is already ranked TR output from another method — this skill consumes raw coordinate BED regions, not TR ranking tables.

## Inputs

- DAR BED files from bulk TR perturbation datasets (DiffBind export.bed, genome build matched: hg38 or mm10) for CTCF, ZBTB7A, FOXA2, RUNX1, and BCL11B
- BIT pre-processed TR ChIP-seq reference library (over 10,000 human and over 5,000 mouse TR ChIP-seq datasets)
- BART data libraries (hg38_library / mm10_library tar.gz)
- Motif references: ENCODE (i-cisTarget), JASPAR/TRANSFAC (WhichTF, HOMER)

## Outputs

- Per-method ranked TR tables (BIT output includes BIT_score and Rank columns; BART output *_bart_results.txt ranks all TRs)
- Per-method MRR values computed as mean reciprocal rank over the five perturbed TRs
- mrr_benchmark.csv of per-method MRR values
- Fig. 3f-style bar chart of MRR per method

## How to apply

Obtain differentially accessible region BED files (e.g., RUNX1 cKO workflow: DiffBind report generation with dba.blacklist mm10 removal, dba.count minOverlap=2, MACS2 q 0.01 peaks, export.bed split by Fold > 0 / Fold < 0) for the perturbed TRs CTCF, ZBTB7A, FOXA2, RUNX1, and BCL11B, using the same genome build as the reference library. Run BIT on each BED file to produce TR ranking tables containing BIT_score and Rank columns, then run comparison methods on the same inputs: BART and ChIP-Atlas (ChIP-seq binding-based), i-cisTarget (motif-based with ENCODE references), and WhichTF and HOMER (motif-based with JASPAR/TRANSFAC references). For each method and each of the five perturbed TRs, extract the rank of the perturbed TR from the method's ranked output, compute the reciprocal rank (1/rank), and average over the five TRs to obtain the per-method MRR. Write per-method MRR values to mrr_benchmark.csv and visualize as a Fig. 3f-style bar chart; validation is reproduction of the reported result where BIT attains the highest MRR among the six methods.

## Related tools

- **BIT** (Primary Bayesian method: transforms query peak coordinates to fixed-length binary vectors, fits a Bayesian hierarchical model via Gibbs-sampler MCMC, and assigns each TR a BIT_score used for ranking) — https://github.com/ZeyuL01/BIT
- **BART** (ChIP-seq binding-based comparison method run in region mode on the same scored BED input) — https://github.com/zanglab/bart2
- **ChIP-Atlas** (ChIP-seq-based comparison method that ranks TRs using p values from Fisher's exact tests)
- **I-cisTarget** (Motif-based comparison method using ENCODE references and recovery analysis)
- **WhichTF** (Motif-based comparison method using JASPAR/TRANSFAC references and hypergeometric tests)
- **HOMER** (Motif-based comparison method leveraging TR binding motifs as reference)
- **R** (Language of the BIT R package (GitHub) used to run the method and compute MRR; R DiffBind used to generate DAR BED inputs) — https://github.com/ZeyuL01/BIT

## Examples

```
bart2 region -i query_DAR.bed -c 4 -s hg38 --outdir bart2_output
```

## Evaluation signals

- Each method's ranked output must contain an extractable rank for each of the five perturbed TRs (CTCF, ZBTB7A, FOXA2, RUNX1, BCL11B); missing TRs invalidate the benchmark.
- MRR for each method must equal the mean of 1/rank over exactly five perturbed TRs; recompute independently and check the average.
- Per-method MRR values in mrr_benchmark.csv must be checkable against the published Fig. 3f values.
- Reproduction succeeds if BIT attains the highest MRR among the six methods (BIT, BART, ChIP-Atlas, i-cisTarget, WhichTF, HOMER), as reported in Fig. 3f.
- BIT output tables must contain BIT_score and Rank columns, and all methods must be run on the same DAR BED inputs with the same genome build and underlying epigenomic profiling datasets.

## Limitations

- Sequencing biases such as GC content bias (e.g., Illumina platform) or PCR amplification bias can reduce the reliability of the input epigenomic region set and the TR binding profiles.
- Biological complexities of TR regulation — dynamic TR binding across cell types and time, chromatin structure and epigenetic modifications affecting affinity/specificity, and cooperative binding — can reduce BIT's performance.
- The number of parameters BIT infers grows with ChIP-seq library size; analyzing dozens or hundreds of epigenomic profiling datasets with the Gibbs sampler can become increasingly time-consuming.
- Jaccard similarity among the TR-identification approaches is generally low (Supplementary Fig. 9), so methods may disagree on top-ranked TRs; comparisons reference supplementary figures not always present in the main text.

## Evidence

- [readme] BIT reference library scale: [evidence span withheld — anchor status: external_doc]
- [readme] BART region-mode invocation on BED input: [evidence span withheld — anchor status: not_found]
