---
name: beta-binomial-enrichment-testing-with-gc-overdispersion-correction
description: Use when detecting significant RBP binding to annotated repetitive elements
  or transcriptomic windows from eCLIP data with matched inputs, especially multi-mapping/repeat
  targets that peak callers miss.
when_to_use_negative:
- CLIP datasets lacking a matched input sample — Skipper requires matched input to
  construct the beta-binomial model and cannot analyze them
- Analyses where only peak-called binding sites are needed and multi-mapping repeat/multi-mapping
  sequences are not of interest
- Datasets not aligned with STAR or missing the RepeatMasker annotation files Skipper
  expects as REPEAT_TABLE/REPEAT_BED
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3524
tools:
- name: Skipper
  role: End-to-end Snakemake workflow performing beta-binomial enrichment testing
    of IP vs. input reads over windows and RepeatMasker elements with GC and overdispersion
    correction
  repo: github.com/YeoLab/skipper
- name: Snakemake 9.12.0
  role: Workflow engine used to execute the Skipper pipeline (conda environment snakemake9)
- name: STAR
  role: Aligner used for the genome index and eCLIP read alignment feeding the enrichment
    tests
- name: SLURM
  role: HPC executor (snakemake-executor-plugin-slurm 1.4.0) used to parallelize Skipper
    jobs across cluster nodes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:aef217492d73f9583fad912f07f3d89d4bd9c0279529dee92ad7a3f2c1fdf77b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  tools:
  - SLURM
  - STAR
  - Skipper
  - Snakemake 9.12.0
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# beta-binomial-enrichment-testing-with-gc-overdispersion-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skipper's beta-binomial statistical framework tests STAR-aligned eCLIP IP vs. matched input reads for enrichment at annotated features (windows and RepeatMasker repetitive elements), correcting for GC content and overdispersion so that multi-mapping repeat targets can be detected across 219 ENCODE eCLIP datasets.

## When to use

Use when detecting significant RBP binding to annotated repetitive elements or transcriptomic windows from eCLIP data with matched inputs, especially multi-mapping/repeat targets that peak callers miss.

## When NOT to use

- CLIP datasets lacking a matched input sample — Skipper requires matched input to construct the beta-binomial model and cannot analyze them
- Analyses where only peak-called binding sites are needed and multi-mapping repeat/multi-mapping sequences are not of interest
- Datasets not aligned with STAR or missing the RepeatMasker annotation files Skipper expects as REPEAT_TABLE/REPEAT_BED

## Inputs

- STAR-aligned eCLIP IP BAM files with matched input control replicates (ENCODE enhanced CLIP)
- RepeatMasker repetitive element coordinates from UCSC Table Browser
- REPEAT_TABLE and REPEAT_BED annotation files (chr,start,end,label,score,strand,name,class,family,proportion_gc)
- GENCODE/Ensembl GFF3 annotation (e.g., gencode.v49.basic.annotation.gff3) and GRCh38 genome FASTA

## Outputs

- Per-repeat-element enrichment test results (beta-binomial significance per element)
- repeat_enrichment_summary.tsv with the fraction of experiments having at least one significantly enriched repeat
- Clusters of experiments by enriched simple-repeat classes

## How to apply

Generate a gzipped, sorted, non-overlapping repeat annotation file (chr,start,end,label,score,strand,name,class,family,proportion_gc) from UCSC Table Browser RepeatMasker coordinates; Skipper auto-generates this as REPEAT_BED. Run the Skipper Snakemake 9.12.0 workflow (via SLURM executor plugin) on STAR-aligned ENCODE eCLIP IP and matched input replicates, specifying REPEAT_TABLE and REPEAT_BED so reads aggregate per repeat element alongside the tiled-window analysis. Test each element with the beta-binomial framework using NORMALIZATION_MODE='new' (pseudocount-based), OVERDISPERSION_MODE='input', GINI_CUTOFF=0.9 to remove narrow 'skyscraper' windows, a configured THRESHOLD_MIN, and DEFINITION_OF_REPRODUCIBILITY='ALL'. Tabulate the fraction of enhanced CLIP experiments with at least one significantly enriched repeat element into repeat_enrichment_summary.tsv and cluster experiments by enriched simple-repeat classes to check motif patterns (GU-rich dominance, QKI-(UAC)n, RBFOX2-UGUG, MATR3/SAFB2 LINE1 strand/age strata).

## Related tools

- **Skipper** (End-to-end Snakemake workflow performing beta-binomial enrichment testing of IP vs. input reads over windows and RepeatMasker elements with GC and overdispersion correction) — github.com/YeoLab/skipper
- **Snakemake 9.12.0** (Workflow engine used to execute the Skipper pipeline (conda environment snakemake9))
- **STAR** (Aligner used for the genome index and eCLIP read alignment feeding the enrichment tests)
- **SLURM** (HPC executor (snakemake-executor-plugin-slurm 1.4.0) used to parallelize Skipper jobs across cluster nodes)

## Examples

```
conda create -n snakemake9 snakemake=9.12.0; conda activate snakemake9; snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Summary table reproduces the reported statistic that bound repeat elements are identified for 99% of enhanced CLIP experiments (216 of 219 ENCODE eCLIP datasets)
- Simple-repeat clustering recovers the reported patterns: GU-rich elements dominate, QKI recovers (UAC)n, and RBFOX2 recovers UGUG-containing motifs
- MATR3 and SAFB2 enriched LINE1 elements, stratified by strand and element age, show the reported strand/age patterns
- REPEAT_BED file is gzipped, sorted, and non-overlapping with the 10-column (chr,start,end,label,score,strand,name,class,family,proportion_gc) schema
- Deviations from the reported 216/219 recovery rate or motif-cluster patterns beyond reported values indicate a failed reproduction

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be analyzed
- Users can blacklist reproducible regions of their choosing, but further work on best practices for blacklisting is warranted
- Threshold parameters (GINI_CUTOFF=0.9, THRESHOLD_MIN) and reproducibility definition must be configured; defaults affect sensitivity to narrow 'skyscraper' windows

## Evidence
