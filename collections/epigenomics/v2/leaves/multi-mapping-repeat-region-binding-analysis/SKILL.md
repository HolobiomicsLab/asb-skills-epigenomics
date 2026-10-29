---
name: multi-mapping-repeat-region-binding-analysis
description: Use when testing CLIP-seq/eCLIP data for enrichment at multi-mapping
  annotated repetitive elements (RepeatMasker), where peak callers fail and RBPs may
  bind simple repeats such as GU-rich motifs.
when_to_use_negative:
- The eCLIP dataset lacks a matched input sample, since Skipper requires one to construct
  the beta-binomial model for statistical testing
- The goal is standard transcriptomic peak-level binding site calling, not multi-mapping/repeat
  element testing (use the default Skipper window analysis instead)
- The repeat annotation BED has not been cleaned (overlapping elements, unsorted coordinates),
  since Skipper auto-generates REPEAT_BED expecting non-overlapping sorted entries
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0080
tools:
- name: Skipper
  role: End-to-end Snakemake workflow that aggregates read-start signal per annotated
    repeat element and tests enrichment with a GC-content-aware beta-binomial model
  repo: https://github.com/YeoLab/skipper
- name: Snakemake 9.12.0
  role: Workflow engine used to execute the Skipper pipeline in the snakemake9 conda
    environment
  repo: https://github.com/YeoLab/skipper
- name: snakemake-executor-plugin-slurm 1.4.0
  role: SLURM cluster executor for parallelizing Skipper jobs across HPC compute nodes
  repo: https://github.com/YeoLab/skipper
- name: STAR
  role: Aligns eCLIP IP and matched input reads to the GRCh38 genome index built by
    Skipper
  repo: https://github.com/YeoLab/skipper
- name: SLURM
  role: HPC workload manager on which the Skipper repeat-enrichment run is scheduled
  repo: https://github.com/YeoLab/skipper
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:6e45d589a7689fd34df796dd5c11de3cef5fda6cb7397e22429ecfbb23d9645d
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
  - Snakemake 9.12.0
  - snakemake-executor-plugin-slurm 1.4.0
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# multi-mapping-repeat-region-binding-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect significant RBP binding to annotated repetitive elements in eCLIP data using Skipper's beta-binomial framework with GC-content correction and input-based overdispersion modeling, aggregating reads per RepeatMasker element rather than calling peaks.

## When to use

Use when testing CLIP-seq/eCLIP data for enrichment at multi-mapping annotated repetitive elements (RepeatMasker), where peak callers fail and RBPs may bind simple repeats such as GU-rich motifs.

## When NOT to use

- The eCLIP dataset lacks a matched input sample, since Skipper requires one to construct the beta-binomial model for statistical testing
- The goal is standard transcriptomic peak-level binding site calling, not multi-mapping/repeat element testing (use the default Skipper window analysis instead)
- The repeat annotation BED has not been cleaned (overlapping elements, unsorted coordinates), since Skipper auto-generates REPEAT_BED expecting non-overlapping sorted entries

## Inputs

- STAR-aligned eCLIP IP BAM files with matched input/control samples
- RepeatMasker repeat annotations from UCSC Table Browser
- Skipper REPEAT_BED file (chr,start,end,label,score,strand,name,class,family,proportion_gc)
- Skipper config YAML with REPEAT_TABLE and REPEAT_BED entries
- Manifest CSV of sample identifiers and file paths

## Outputs

- Per-repeat-element enrichment statistics from the beta-binomial test
- repeat_enrichment_summary.tsv with the fraction of experiments having significantly enriched repeat elements
- Clusters of experiments grouped by enriched simple-repeat classes
- Strand/age-stratified LINE1 enrichment tables for specific RBPs

## How to apply

Obtain repetitive element coordinates from the UCSC Table Browser (RepeatMasker) and build the gzipped, sorted, non-overlapping repeat annotation BED (chr,start,end,label,score,strand,name,class,family,proportion_gc) that Skipper uses as REPEAT_BED, supplied together with REPEAT_TABLE in the Skipper config. Run the Skipper Snakemake workflow (Snakemake 9.12.0, SLURM executor plugin 1.4.0) on STAR-aligned ENCODE eCLIP IP samples with matched input controls so reads are aggregated per repeat element alongside the transcript window analysis. Test each annotated repeat element with the beta-binomial model using NORMALIZATION_MODE='new' (pseudocount-based), OVERDISPERSION_MODE='input', GINI_CUTOFF=0.9 to remove skyscraper windows, configured THRESHOLD_MIN, and DEFINITION_OF_REPRODUCIBILITY='ALL'. Tabulate the fraction of enhanced CLIP experiments with at least one significantly enriched repeat, cluster experiments by enriched simple-repeat classes, and for RBPs like MATR3/SAFB2 stratify enriched LINE1 elements by strand and element age.

## Related tools

- **Skipper** (End-to-end Snakemake workflow that aggregates read-start signal per annotated repeat element and tests enrichment with a GC-content-aware beta-binomial model) — https://github.com/YeoLab/skipper
- **Snakemake 9.12.0** (Workflow engine used to execute the Skipper pipeline in the snakemake9 conda environment) — https://github.com/YeoLab/skipper
- **snakemake-executor-plugin-slurm 1.4.0** (SLURM cluster executor for parallelizing Skipper jobs across HPC compute nodes) — https://github.com/YeoLab/skipper
- **STAR** (Aligns eCLIP IP and matched input reads to the GRCh38 genome index built by Skipper) — https://github.com/YeoLab/skipper
- **SLURM** (HPC workload manager on which the Skipper repeat-enrichment run is scheduled) — https://github.com/YeoLab/skipper

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- The summary table reproduces the reported statistic that bound repeat elements are identified for 99% (216/219) of enhanced CLIP experiments
- Simple-repeat clustering recovers the reported GU-rich-dominant pattern across experiments
- QKI recovers (UAC)n motifs and RBFOX2 recovers UGUG-containing motifs among enriched repeats
- For MATR3 and SAFB2, enriched LINE1 elements show the reported strand and element-age patterns
- The repeat BED validates as gzipped, sorted, and non-overlapping before the Snakemake run starts

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be analyzed
- GINI_CUTOFF=0.9 filtering removes narrow 'skyscraper' windows, which may exclude some genuine high-intensity repeat binding
- Best practices for blacklisting reproducible regions are not established and further work is needed

## Evidence

- [methods] A filter used to remove windows with incredibly narrow peaks (skyscrapers): [evidence span withheld — anchor status: not_found]
