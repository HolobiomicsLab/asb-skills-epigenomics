---
name: rbp-binding-site-count-aggregation
description: Use when comparing the number of enriched candidate binding sites called
  by Skipper against Piranha and CLIPper on matched eCLIP replicates, or when benchmarking
  a new CLIP-seq caller's sensitivity on ENCODE data.
when_to_use_negative:
- CLIP datasets lacking a matched input sample, since Skipper requires one to construct
  the beta-binomial model for statistical testing
- Datasets where the goal is motif discovery or binding-affinity estimation rather
  than caller sensitivity benchmarking
- Analyses targeting repetitive-element or multi-mapping-sequence binding, which this
  peak-count comparison framework does not address
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Skipper
  role: End-to-end Snakemake eCLIP pipeline producing reproducible enriched windows
    via transcript-tiling instead of peak calling
  repo: https://github.com/YeoLab/skipper
- name: Piranha
  role: Alternative peak caller run on the same aligned eCLIP replicates and window
    framework for count comparison
- name: CLIPper
  role: Source of the previously reported ENCODE peak calls intersected by genomic
    interval overlap
  repo: https://github.com/YeoLab/clipper
- name: STAR
  role: Aligner whose genome index is built from the GENCODE GFF and genome FASTA
    before caller benchmarking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:86e5f8b7f908c4ca3ba38042773b4bbcf8dc227a320c591970d3ed93b9dec98b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# rbp-binding-site-count-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark-style aggregation of enriched binding-site counts across CLIP-seq callers: run Skipper (window-tiling, peak-free) side-by-side with Piranha and CLIPper on matched ENCODE eCLIP IP/input replicates, then intersect and summarize per-RBP site counts to quantify how many more transcriptomic binding sites Skipper recovers.

## When to use

Use when comparing the number of enriched candidate binding sites called by Skipper against Piranha and CLIPper on matched eCLIP replicates, or when benchmarking a new CLIP-seq caller's sensitivity on ENCODE data.

## When NOT to use

- CLIP datasets lacking a matched input sample, since Skipper requires one to construct the beta-binomial model for statistical testing
- Datasets where the goal is motif discovery or binding-affinity estimation rather than caller sensitivity benchmarking
- Analyses targeting repetitive-element or multi-mapping-sequence binding, which this peak-count comparison framework does not address

## Inputs

- Matched IP and input eCLIP FASTQ replicates for benchmark RBPs from ENCODE
- GENCODE annotation GFF3 (e.g. gencode.v49.basic.annotation.gff3) and GRCh38 genome FASTA for the STAR index
- Previously reported ENCODE CLIPper peak calls (BED) for interval overlap

## Outputs

- Skipper reproducible_enriched_windows outputs per RBP
- Piranha peak calls over the identical tiled window framework
- method_comparison_counts.csv (per-RBP peak counts for Skipper, Piranha, CLIPper and fraction of RBPs where Skipper wins)
- Figure 2A ternary plot and Figure 2B scatter summaries of the count comparison

## How to apply

Download matched IP and input eCLIP replicates for benchmark RBPs from the ENCODE project portal (219 datasets in the paper). Run the Skipper end-to-end Snakemake pipeline — preprocessing and trimming, STAR alignment using a genome index built from a GENCODE GFF (e.g. gencode.v49.basic.annotation.gff3) and GRCh38 genome FASTA, then window-based enriched-site calling — to obtain reproducible_enriched_windows outputs. Run Piranha on the same aligned replicates over the identical tiled window framework, and intersect Skipper and Piranha coordinates with prior ENCODE CLIPper peak calls via genomic interval overlap to assign shared versus unique sites per RBP. Aggregate per-RBP counts for the three callers and compute the fraction of RBPs for which Skipper calls more sites; render Figure 2A ternary and Figure 2B scatter summaries and write method_comparison_counts.csv.

## Related tools

- **Skipper** (End-to-end Snakemake eCLIP pipeline producing reproducible enriched windows via transcript-tiling instead of peak calling) — https://github.com/YeoLab/skipper
- **Piranha** (Alternative peak caller run on the same aligned eCLIP replicates and window framework for count comparison)
- **CLIPper** (Source of the previously reported ENCODE peak calls intersected by genomic interval overlap) — https://github.com/YeoLab/clipper
- **STAR** (Aligner whose genome index is built from the GENCODE GFF and genome FASTA before caller benchmarking)

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Reproduce the reported averages: Skipper ~21,310 reproducible enriched windows, Piranha ~5,039, CLIPper ~6,904 peaks across 219 ENCODE eCLIP datasets
- Confirm Skipper exceeds Piranha/CLIPper for ~72% of RBPs, consistent with 210%-320% more transcriptomic binding sites on average
- Verify Skipper and Piranha peak calls share the identical tiled window framework so counts are directly comparable
- Check interval-overlap assignments partition shared and unique binding sites per RBP without double counting

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets without these controls cannot be analyzed
- Skipper omits peak calling entirely, so counts reflect enriched transcriptomic windows rather than peaks and are not one-to-one with peak-caller outputs
- First Skipper run is slow because it parses the GFF and generates the STAR genome index; subsequent runs are faster

## Evidence
