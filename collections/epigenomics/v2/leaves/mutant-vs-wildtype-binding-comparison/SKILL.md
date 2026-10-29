---
name: mutant-vs-wildtype-binding-comparison
description: Use when eCLIP data exist for both wild-type and missense-mutant versions
  of an RBP (e.g., RPS19 R62W/R101H) and the goal is to detect altered binding — region-proportion
  shifts or novel target classes like tRNA loci.
when_to_use_negative:
- CLIP datasets lacking matched input samples — Skipper cannot construct the beta-binomial
  model, so mutant-vs-WT enrichment cannot be tested
- Questions about the mechanism of binding changes (complex formation, localization,
  vs. binding preference) — eCLIP data alone cannot distinguish these
- Single-replicate data without matched WT/mutant pairs — paired t-tests across matched
  replicates are required for the differential enrichment step
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0099
tools:
- name: Skipper
  role: End-to-end eCLIP pipeline (preprocessing, alignment with STAR, GFF partitioning,
    enriched window identification) that produces the reproducible_enriched_windows
    tables compared between WT and mutant samples
  repo: https://github.com/YeoLab/skipper
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:e878a253fd92141fd36c5fdcdcb1b0ca640ce42b0c229a596ba8fa647a8d908d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  repo_url: https://github.com/YeoLab/skipper
  tools:
  - Skipper
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mutant-vs-wildtype-binding-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compare eCLIP RNA-binding profiles of mutant versus wild-type RBP constructs using Skipper's reproducible_enriched_windows: quantify shifts in transcript-region binding proportions and test differential enrichment (e.g., tRNA loci) with paired statistics across matched replicates.

## When to use

Use when eCLIP data exist for both wild-type and missense-mutant versions of an RBP (e.g., RPS19 R62W/R101H) and the goal is to detect altered binding — region-proportion shifts or novel target classes like tRNA loci.

## When NOT to use

- CLIP datasets lacking matched input samples — Skipper cannot construct the beta-binomial model, so mutant-vs-WT enrichment cannot be tested
- Questions about the mechanism of binding changes (complex formation, localization, vs. binding preference) — eCLIP data alone cannot distinguish these
- Single-replicate data without matched WT/mutant pairs — paired t-tests across matched replicates are required for the differential enrichment step

## Inputs

- Skipper reproducible_enriched_windows tables for WT and mutant eCLIP replicates (e.g., WT, R62W, R101H RPS19)
- Annotated transcript-region assignments (feature_type_top from Skipper's partitioned GENCODE/Ensembl GFF)
- Annotated tRNA locus coordinates for window subsetting
- Raw eCLIP FASTQ files for WT and mutant constructs with matched input controls

## Outputs

- Per-sample feature-proportion table (fraction of reproducible enriched windows per transcript region)
- WT-to-mutant region-shift comparison (e.g., CDS 33%->45%, deep intron 20%->7%) rendered as figure7e_feature_proportions.pdf
- Per-replicate tRNA enrichment values with paired t-test p-values rendered as figure7f_trna_enrichment.pdf

## How to apply

Run Skipper separately on WT and mutant RPS19 eCLIP replicates, then load each sample's reproducible_enriched_windows table. Aggregate windows by feature_type_top within each sample to compute the fraction of reproducible enriched windows per transcript region (CDS, deep introns, UTRs), and quantify the WT-to-mutant shift — for RPS19 mutants this reproduces the CDS increase from 33% to 45% and deep-intron decrease from 20% to 7% (p < 2.2e−16). Next, subset windows mapping to annotated tRNA loci, compute per-replicate tRNA enrichment, and apply paired t-tests across matched WT/mutant replicates to test whether mutants show significantly increased tRNA binding. Render the comparison as region-proportion and tRNA-enrichment figures (e.g., figure7e_feature_proportions.pdf and figure7f_trna_enrichment.pdf). Rationale: Skipper's window-tiling statistical framework calls far more transcriptomic and repetitive-element binding sites than peak calling, giving the sensitivity needed to detect mutant-specific binding redistribution.

## Related tools

- **Skipper** (End-to-end eCLIP pipeline (preprocessing, alignment with STAR, GFF partitioning, enriched window identification) that produces the reproducible_enriched_windows tables compared between WT and mutant samples) — https://github.com/YeoLab/skipper

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Computed feature proportions reproduce the reported CDS shift from 33% to 45% and deep-intron shift from 20% to 7% between WT and mutant RPS19
- Paired t-tests yield significant tRNA enrichment p-values for both R62W and R101H mutants relative to matched WT replicates
- WT-to-mutant region-proportion differences achieve p < 2.2e−16 on the aggregated feature_type_top fractions
- Each sample's window fractions sum to 1 across feature_type_top categories of reproducible enriched windows
- Mutant and WT replicates are matched pairwise (same biological design) before applying paired t-tests

## Limitations

- RPS19 follow-up relied on expressing variant ORFs in HEK293T cells, which may not reflect the true disease state
- eCLIP data alone do not answer whether observed binding changes are due to differences in complex formation, localization, binding preference, or another mechanism
- Skipper requires a matched input sample; CLIP datasets lacking these controls cannot be analyzed

## Evidence

- [results] Both mutant RPS19 constructs (R62W, R101H) induce large-scale changes in the RNA interactome: coding sequence binding rises from 33% to 45% while deep intronic binding falls from 20% to 7% (p < 2.2e−16).: [evidence span withheld — anchor status: not_found]
