---
name: trna-binding-enrichment-testing
description: Use when Skipper reproducible_enriched_windows tables exist for matched
  WT and mutant eCLIP replicates and the goal is to quantify and statistically test
  binding shifts toward annotated tRNA (or other repeat/ncRNA) loci.
when_to_use_negative:
- CLIP datasets lack matched input samples, since Skipper cannot construct its beta-binomial
  model and thus no reproducible enriched windows exist to intersect with tRNA loci
- The question is which transcript regions (CDS vs introns vs UTRs) windows fall in
  rather than testing enrichment at tRNA loci specifically — use a feature-proportion
  aggregation (figure7e-style) instead
- Only single replicates per condition are available, so a paired t-test across matched
  replicate pairs is not statistically valid
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3524
tools:
- name: Skipper
  role: End-to-end eCLIP pipeline producing reproducible_enriched_windows tables and
    feature annotations that are subset to tRNA loci for enrichment testing
  repo: github.com/YeoLab/skipper
- name: STAR
  role: Aligns preprocessed eCLIP reads to the genome as the first stage of the Skipper
    workflow feeding window identification
- name: Snakemake 9.12.0
  role: Workflow engine used to execute the Skipper pipeline that generates the input
    windows
  repo: github.com/YeoLab/skipper
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:67e0ae86ea277c319c2d03674ac794ffa84341998a85fec29be7f271898b9a2c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  tools:
  - STAR
  - Skipper
  - Snakemake 9.12.0
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# trna-binding-enrichment-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tests whether an RBP eCLIP experiment shows significant enrichment of reproducible Skipper binding windows over annotated tRNA loci, and compares tRNA enrichment between matched wild-type and mutant replicate pairs with paired t-tests. Used to detect mutant-induced retargeting (e.g., RPS19 R62W/R101H) toward non-coding RNAs.

## When to use

Use when Skipper reproducible_enriched_windows tables exist for matched WT and mutant eCLIP replicates and the goal is to quantify and statistically test binding shifts toward annotated tRNA (or other repeat/ncRNA) loci.

## When NOT to use

- CLIP datasets lack matched input samples, since Skipper cannot construct its beta-binomial model and thus no reproducible enriched windows exist to intersect with tRNA loci
- The question is which transcript regions (CDS vs introns vs UTRs) windows fall in rather than testing enrichment at tRNA loci specifically — use a feature-proportion aggregation (figure7e-style) instead
- Only single replicates per condition are available, so a paired t-test across matched replicate pairs is not statistically valid

## Inputs

- Skipper reproducible_enriched_windows tables for WT, R62W, and R101H RPS19 eCLIP replicates (plus matched input controls)
- Partitioned GENCODE or Ensembl GFF annotation containing tRNA loci and other feature annotations
- Genome FASTA (e.g., GRCh38.primary_assembly.genome.fa) and STAR genome index used by Skipper

## Outputs

- Per-replicate tRNA enrichment values for each WT/mutant sample
- Paired t-test statistics and p-values comparing mutant vs WT tRNA enrichment across replicate pairs
- Significance-annotated comparison plot (e.g., figure7f_trna_enrichment.pdf)

## How to apply

First run the Skipper end-to-end pipeline (preprocessing, STAR alignment, GFF partitioning, enriched window identification, fine-mapping) on each WT and mutant eCLIP replicate, requiring matched input controls for the beta-binomial model. Load each replicate's reproducible_enriched_windows table and intersect the reproducible enriched windows with annotated tRNA loci (from the partitioned GENCODE/Ensembl GFF or repeat annotations). Compute a per-replicate tRNA enrichment metric (e.g., fraction of reproducible windows mapping to tRNA loci, normalized against genomic background). Then apply paired t-tests across matched WT/mutant replicate pairs to test whether mutants show significantly increased tRNA binding, and render the comparison with paired-test significance annotations (as in figure7f_trna_enrichment.pdf).

## Related tools

- **Skipper** (End-to-end eCLIP pipeline producing reproducible_enriched_windows tables and feature annotations that are subset to tRNA loci for enrichment testing) — github.com/YeoLab/skipper
- **STAR** (Aligns preprocessed eCLIP reads to the genome as the first stage of the Skipper workflow feeding window identification)
- **Snakemake 9.12.0** (Workflow engine used to execute the Skipper pipeline that generates the input windows) — github.com/YeoLab/skipper

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Each replicate's reproducible_enriched_windows table exists and windows can be unambiguously assigned to tRNA vs non-tRNA annotations
- Per-replicate tRNA enrichment values are computed for all matched WT and mutant replicates with no missing pairs before running the paired t-test
- Paired t-tests yield significant p-values for both mutants (R62W and R101H) relative to WT for increased tRNA binding
- tRNA enrichment results are consistent with the independently computed feature-proportion shift (CDS rising 33%->45%, deep intronic falling 20%->7%, p < 2.2e-16) reported in the companion analysis
- The rendered figure7f shows paired significance annotations matching the computed t-test results

## Limitations

- eCLIP data alone cannot determine whether observed tRNA-binding changes arise from differences in complex formation, localization, or binding preference
- Mutant ORFs expressed in HEK293T cells may not reflect the true Diamond-Blackfan disease state, so tRNA retargeting may not generalize to patient biology
- Skipper's window detection depends on a matched input sample; without it, no reproducible enriched windows (and thus no tRNA enrichment test) are possible

## Evidence
