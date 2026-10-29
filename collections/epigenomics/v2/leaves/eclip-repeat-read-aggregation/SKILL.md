---
name: eclip-repeat-read-aggregation
description: Use when eCLIP/eCLIP-seq IP and matched input samples must be tested
  for significant binding to annotated repetitive elements (e.g., GU-rich simple repeats,
  LINE1) rather than annotated transcripts.
when_to_use_negative:
- The CLIP dataset lacks a matched input sample, since Skipper requires one to construct
  the beta-binomial model for statistical testing
- The goal is peak calling on annotated transcripts rather than testing pre-annotated
  multi-mapping repetitive elements
- The alignment has not been done or the repeat annotation file has overlapping intervals,
  since REPEAT_BED must be sorted, non-overlapping, and gzip-compressed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Skipper
  role: End-to-end workflow that aggregates reads per RepeatMasker element and applies
    the beta-binomial enrichment test with GC-content and overdispersion modeling
  repo: https://github.com/YeoLab/skipper
- name: Snakemake
  role: Workflow engine (v9.12.0) used to run Skipper with REPEAT_TABLE and REPEAT_BED
    inputs
  repo: https://github.com/YeoLab/skipper
- name: STAR
  role: Aligns eCLIP reads to the genome index before per-repeat read aggregation
- name: SLURM
  role: HPC executor plugin (snakemake-executor-plugin-slurm=1.4.0) used to parallelize
    Skipper jobs across cluster nodes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:9637ce88f022c5b5ecbf9dbda34d0aeb4f207e3e24112a7193edc54069c7ad83
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
  - SLURM
  - STAR
  - Skipper
  - Snakemake
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# eclip-repeat-read-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregate eCLIP reads per RepeatMasker-annotated repetitive element and test each element for enrichment using Skipper's beta-binomial framework with GC-content and overdispersion modeling, enabling detection of RBP binding to multi-mapping repeat sequences that peak callers miss.

## When to use

Use when eCLIP/eCLIP-seq IP and matched input samples must be tested for significant binding to annotated repetitive elements (e.g., GU-rich simple repeats, LINE1) rather than annotated transcripts.

## When NOT to use

- The CLIP dataset lacks a matched input sample, since Skipper requires one to construct the beta-binomial model for statistical testing
- The goal is peak calling on annotated transcripts rather than testing pre-annotated multi-mapping repetitive elements
- The alignment has not been done or the repeat annotation file has overlapping intervals, since REPEAT_BED must be sorted, non-overlapping, and gzip-compressed

## Inputs

- STAR-aligned eCLIP IP replicate BAM files (e.g., ENCODE enhanced CLIP)
- Matched input (size-matched) control replicate BAM files
- RepeatMasker repeat annotation BED with 10 columns (chr,start,end,label,score,strand,name,class,family,proportion_gc), gzipped, sorted, non-overlapping (REPEAT_BED)
- Repeat annotation table (REPEAT_TABLE)
- Skipper config YAML and sample manifest CSV

## Outputs

- Per-experiment tables of significantly enriched repeat elements with beta-binomial test statistics
- repeat_enrichment_summary.tsv reporting the fraction of enhanced CLIP experiments with ≥1 enriched repeat
- Simple-repeat clustering assignment of experiments by enriched repeat class (for motif-pattern verification)

## How to apply

Obtain repetitive element coordinates from the UCSC Table Browser (RepeatMasker) and generate the gzipped, sorted, non-overlapping repeat annotation file Skipper auto-detects as REPEAT_BED with columns chr,start,end,label,score,strand,name,class,family,proportion_gc. Run the Skipper Snakemake workflow on STAR-aligned ENCODE eCLIP IP and matched input replicates, supplying REPEAT_TABLE and REPEAT_BED so reads aggregate per RepeatMasker element alongside the window-level analysis. Test each annotated element with the beta-binomial framework using NORMALIZATION_MODE='new' (pseudocount-based), OVERDISPERSION_MODE='input' (so the matched input sample drives overdispersion estimation), GINI_CUTOFF=0.9 to remove skyscraper windows, a configured THRESHOLD_MIN, and DEFINITION_OF_REPRODUCIBILITY='ALL'. Tabulate the fraction of enhanced CLIP experiments with at least one significantly enriched element and validate against the reported 99% (216/219) recovery rate; optionally cluster experiments by enriched simple-repeat classes to recover GU-rich dominance, QKI (UAC)n, and RBFOX2 UGUG motif patterns.

## Related tools

- **Skipper** (End-to-end workflow that aggregates reads per RepeatMasker element and applies the beta-binomial enrichment test with GC-content and overdispersion modeling) — https://github.com/YeoLab/skipper
- **Snakemake** (Workflow engine (v9.12.0) used to run Skipper with REPEAT_TABLE and REPEAT_BED inputs) — https://github.com/YeoLab/skipper
- **STAR** (Aligns eCLIP reads to the genome index before per-repeat read aggregation)
- **SLURM** (HPC executor plugin (snakemake-executor-plugin-slurm=1.4.0) used to parallelize Skipper jobs across cluster nodes)

## Examples

```
snakemake -s Skipper.py --configfile repeat_enrichment_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- The enrichment summary reproduces the reported statistic: bound repeat elements identified for 99% of enhanced CLIP experiments (216 of 219 datasets)
- Simple-repeat clustering recovers the reported pattern that GU-rich elements dominate across experiments
- QKI recovers (UAC)n-containing motifs and RBFOX2 recovers UGUG-containing motifs among enriched repeats
- For MATR3 and SAFB2, enriched LINE1 elements reproduce the reported strand- and age-stratified patterns
- REPEAT_BED passes schema checks: 10 columns (chr,start,end,label,score,strand,name,class,family,proportion_gc), gzipped, sorted, and non-overlapping

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be analyzed
- Reproducible regions of concern must be manually blacklisted by the user; best practices for blacklisting are not established
- Multi-mapping reads and repetitive elements are handled only through the repeat-annotation testing path, not by peak calling approaches

## Evidence
