---
name: line1-strand-and-age-stratification-analysis
description: Use when Skipper has flagged enriched repetitive elements in eCLIP IP-vs-input
  experiments and you need to characterize binding orientation and evolutionary age
  of LINE1 elements for specific RBPs such as MATR3 or SAFB2.
when_to_use_negative:
- The eCLIP dataset lacks a matched input sample, so Skipper cannot construct the
  beta-binomial model for enrichment testing
- The goal is transcriptomic (gene-region) binding site detection rather than multi-mapping
  repetitive-element binding
- Input samples have not yet been run through the Skipper Snakemake pipeline to produce
  per-element enrichment statistics
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0114
- http://edamontology.org/topic_3512
tools:
- name: Skipper
  role: End-to-end workflow that tests each annotated repeat element for enrichment
    with a beta-binomial framework, aggregating reads per RepeatMasker element via
    REPEAT_TABLE and REPEAT_BED
  repo: https://github.com/YeoLab/skipper
- name: Snakemake
  role: Executes the Skipper pipeline (version 9.12.0) that runs the repeat-element
    enrichment mode
  repo: https://github.com/YeoLab/skipper
- name: STAR
  role: Aligns the ENCODE eCLIP IP and matched input replicates whose reads are then
    aggregated per repeat element
- name: SLURM
  role: HPC executor plugin (snakemake-executor-plugin-slurm=1.4.0) used to parallelize
    Skipper jobs on the cluster
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:9eee5d7d39c190772d7629ef0382c87a5629ef310df00c4e0e8854ceedd0a12d
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
  - http://edamontology.org/topic_0114
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# LINE1 strand and age stratification analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies Skipper-detected enriched LINE1 repetitive elements by genomic strand and element age to verify reported RBP binding patterns (e.g., MATR3 and SAFB2 preferential strand/age enrichment) after beta-binomial enrichment testing of RepeatMasker-annotated elements across eCLIP datasets.

## When to use

Use when Skipper has flagged enriched repetitive elements in eCLIP IP-vs-input experiments and you need to characterize binding orientation and evolutionary age of LINE1 elements for specific RBPs such as MATR3 or SAFB2.

## When NOT to use

- The eCLIP dataset lacks a matched input sample, so Skipper cannot construct the beta-binomial model for enrichment testing
- The goal is transcriptomic (gene-region) binding site detection rather than multi-mapping repetitive-element binding
- Input samples have not yet been run through the Skipper Snakemake pipeline to produce per-element enrichment statistics

## Inputs

- Skipper per-element enrichment results from REPEAT_TABLE/REPEAT_BED runs of ENCODE eCLIP IP and matched input replicates (STAR-aligned)
- RepeatMasker LINE1 annotations with strand and divergence-based age from UCSC Table Browser
- List of significantly enriched (enhanced) CLIP experiments per repeat element

## Outputs

- Strand-stratified table of enriched LINE1 elements per RBP (e.g., MATR3, SAFB2)
- Age-stratified table/bins of enriched LINE1 elements per RBP
- Comparison against the paper's reported MATR3/SAFB2 strand/age patterns

## How to apply

First run the Skipper Snakemake workflow in repetitive-element mode by supplying REPEAT_TABLE and REPEAT_BED (a gzipped, sorted, non-overlapping chr,start,end,label,score,strand,name,class,family,proportion_gc file built from UCSC Table Browser RepeatMasker coordinates), so reads are aggregated per RepeatMasker element rather than per transcript window. Test each element with Skipper's beta-binomial framework using NORMALIZATION_MODE='new', OVERDISPERSION_MODE='input', GINI_CUTOFF=0.9, the configured THRESHOLD_MIN, and DEFINITION_OF_REPRODUCIBILITY='ALL'. For the RBP of interest (e.g., MATR3, SAFB2), subset the significant elements to the LINE1 class, then split them by annotated strand and by RepeatMasker-derived element age bins. Rationale: Skipper's GC-content and overdispersion modeling detects binding to multi-mapping repeat sequence that peak callers miss, making strand- and age-resolved patterns interpretable.

## Related tools

- **Skipper** (End-to-end workflow that tests each annotated repeat element for enrichment with a beta-binomial framework, aggregating reads per RepeatMasker element via REPEAT_TABLE and REPEAT_BED) — https://github.com/YeoLab/skipper
- **Snakemake** (Executes the Skipper pipeline (version 9.12.0) that runs the repeat-element enrichment mode) — https://github.com/YeoLab/skipper
- **STAR** (Aligns the ENCODE eCLIP IP and matched input replicates whose reads are then aggregated per repeat element)
- **SLURM** (HPC executor plugin (snakemake-executor-plugin-slurm=1.4.0) used to parallelize Skipper jobs on the cluster)

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- The overall repeat-enrichment summary reproduces the reported statistic that bound repeat elements are identified for 99% (216/219) of enhanced CLIP experiments
- Enriched LINE1 elements for MATR3 and SAFB2 reproduce the paper's reported strand and element-age patterns when stratified
- Simple-repeat clustering of enriched experiments recovers GU-rich-dominant classes plus QKI-(UAC)n and RBFOX2-UGUG motif patterns
- REPEAT_BED input is confirmed as gzipped, sorted, and non-overlapping before running, matching Skipper's expected annotation format
- Deviation from the reported 216/219 recovery rate or the known motif/strand patterns indicates a failed reproduction

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be analyzed
- Few CLIP-seq tools test multi-mapping repeats, so orthogonal validation of strand/age findings may be limited
- Results depend on RepeatMasker age annotations and the GINI_CUTOFF=0.9 skyscraper filter and THRESHOLD_MIN settings chosen

## Evidence

- [intro] few tools test multi-mapping repeats even when they are principal RBP targets: [evidence span withheld — anchor status: not_found]
