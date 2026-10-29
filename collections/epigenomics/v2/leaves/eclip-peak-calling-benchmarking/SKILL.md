---
name: eclip-peak-calling-benchmarking
description: Use when comparing the number of enriched CLIP-seq/eCLIP binding sites
  called by a new caller against established methods (Piranha, CLIPper) on shared
  ENCODE eCLIP datasets.
when_to_use_negative:
- CLIP datasets lacking a matched input sample — Skipper cannot construct its beta-binomial
  model and cannot be analyzed
- The goal is per-read or motif-level characterization rather than comparing counts
  of enriched binding sites across peak-calling methods
- Analysis targets non-transcriptomic genomic regions outside the tiled windows over
  annotated transcripts
edam_operation: http://edamontology.org/operation_3902
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0659
tools:
- name: Skipper
  role: End-to-end eCLIP pipeline that converts unprocessed reads into annotated binding
    sites by tiling windows over annotated transcripts and calling reproducible enriched
    windows
  repo: https://github.com/YeoLab/skipper
- name: Snakemake
  role: Workflow engine (v9.12.0) used to execute the Skipper pipeline, with snakemake-executor-plugin-slurm
    for HPC execution
  repo: https://github.com/YeoLab/skipper
- name: STAR
  role: Aligns preprocessed eCLIP reads using a genome index generated from the GENCODE
    GFF and genome FASTA
- name: Piranha
  role: Alternative peak caller run on the same aligned eCLIP replicates over the
    identical tiled window framework for benchmark comparison
- name: CLIPper
  role: Peak enrichment recognition tool whose previously reported ENCODE peak calls
    serve as the legacy baseline for interval-overlap comparison
  repo: https://github.com/YeoLab/clipper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:40c1686a31c7261b21b4a93a333f1332ba9b591613904cef33e293789e0fb3dd
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
  - CLIPper
  - Piranha
  - STAR
  - Skipper
  - Snakemake
  edam_operation: http://edamontology.org/operation_3902
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# eclip-peak-calling-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the Skipper end-to-end eCLIP workflow against legacy peak callers (Piranha, CLIPper) by counting reproducible enriched binding-site windows per RBP across matched IP/input ENCODE eCLIP replicates, quantifying how many more transcriptomic binding sites Skipper detects.

## When to use

Use when comparing the number of enriched CLIP-seq/eCLIP binding sites called by a new caller against established methods (Piranha, CLIPper) on shared ENCODE eCLIP datasets.

## When NOT to use

- CLIP datasets lacking a matched input sample — Skipper cannot construct its beta-binomial model and cannot be analyzed
- The goal is per-read or motif-level characterization rather than comparing counts of enriched binding sites across peak-calling methods
- Analysis targets non-transcriptomic genomic regions outside the tiled windows over annotated transcripts

## Inputs

- Matched IP and input eCLIP FASTQ replicates from the ENCODE project portal
- GENCODE GFF3 annotation (e.g., gencode.v49.basic.annotation.gff3.gz) filtered to the best transcript per gene
- GENCODE/Ensembl genome FASTA (e.g., GRCh38.primary_assembly.genome.fa) for building the STAR index
- Previously reported ENCODE CLIPper peak calls (BED)

## Outputs

- Skipper reproducible_enriched_windows outputs per RBP
- Piranha peak calls over the identical tiled window framework
- Per-RBP aggregated peak counts for Skipper, Piranha, and CLIPper (method_comparison_counts.csv)
- Figure 2A ternary plot and Figure 2B scatter summaries of the count comparison

## How to apply

Download matched IP and input eCLIP replicates for benchmark RBPs from the ENCODE project portal. Run the Skipper Snakemake pipeline end to end — preprocessing and trimming, alignment with a STAR genome index built from a GENCODE GFF (e.g., gencode.v49.basic.annotation.gff3) and the GRCh38 primary assembly FASTA, then window-based enriched-site calling — collecting its reproducible_enriched_windows outputs. Run Piranha on the same aligned replicates over the identical tiled window framework, and intersect Skipper and Piranha coordinates with previously reported ENCODE CLIPper peaks by genomic interval overlap to assign shared and unique sites per RBP. Aggregate per-RBP counts for all three callers and compute the fraction of RBPs for which Skipper calls more sites; in the source study Skipper averaged ~21,310 reproducible enriched windows versus ~5,039 (Piranha) and ~6,904 (CLIPper), detecting more sites than Piranha for 72% of RBPs — consistent with 210%-320% more transcriptomic binding sites on average.

## Related tools

- **Skipper** (End-to-end eCLIP pipeline that converts unprocessed reads into annotated binding sites by tiling windows over annotated transcripts and calling reproducible enriched windows) — https://github.com/YeoLab/skipper
- **Snakemake** (Workflow engine (v9.12.0) used to execute the Skipper pipeline, with snakemake-executor-plugin-slurm for HPC execution) — https://github.com/YeoLab/skipper
- **STAR** (Aligns preprocessed eCLIP reads using a genome index generated from the GENCODE GFF and genome FASTA)
- **Piranha** (Alternative peak caller run on the same aligned eCLIP replicates over the identical tiled window framework for benchmark comparison)
- **CLIPper** (Peak enrichment recognition tool whose previously reported ENCODE peak calls serve as the legacy baseline for interval-overlap comparison) — https://github.com/YeoLab/clipper

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Skipper average reproducible enriched windows per dataset ≈ 21,310 across 219 ENCODE eCLIP datasets
- Piranha average ≈ 5,039 and CLIPper average ≈ 6,904 peaks on the same window framework
- Skipper exceeds Piranha/CLIPper for ~72% of RBPs, matching the paper's claim of 210%-320% more transcriptomic binding sites on average
- method_comparison_counts.csv reproduces per-RBP counts consistent with Figure 2A ternary and Figure 2B scatter summaries
- Skipper identifies bound elements for 99% of enhanced CLIP experiments (including annotated repetitive elements)

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be analyzed
- Peak calling approaches are susceptible to false negatives (e.g., intronic signal masked by exonic reads at exon-intron boundaries) and bias against low-abundance transcripts, which shapes benchmark interpretation
- Blacklisting practices for reproducible enriched windows are not established; users must choose regions to blacklist themselves

## Evidence

- [results] Across 219 ENCODE eCLIP datasets, Skipper called an average of 21,310 reproducible enriched windows: [evidence span withheld — anchor status: not_found]
- [readme] CLIPper usage: [evidence span withheld — anchor status: external_doc]
