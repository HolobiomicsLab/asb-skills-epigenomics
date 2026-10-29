---
name: ternary-and-scatter-method-comparison-visualization
description: Use when you have per-RBP enriched-window/peak counts from two or more
  CLIP-seq callers (e.g., Skipper, Piranha, CLIPper) on the same ENCODE eCLIP datasets
  and must visualize and quantify how caller sensitivity compares.
when_to_use_negative:
- CLIP datasets lacking matched input controls — Skipper cannot construct its beta-binomial
  model, so the Skipper arm of the comparison cannot be computed
- Datasets where callers were run on different window tiling or genome builds — genomic
  interval overlap comparisons would be confounded and counts non-comparable
- Analyses not benchmarking caller sensitivity, e.g., motif discovery or fine-mapping
  of individual binding sites
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0769
tools:
- name: Skipper
  role: End-to-end eCLIP pipeline producing reproducible enriched windows; the caller
    whose sensitivity is benchmarked (average ~21,310 windows)
  repo: https://github.com/YeoLab/skipper
- name: Piranha
  role: Alternative peak caller run on the same aligned eCLIP replicates over the
    identical tiled window framework for the count comparison (average ~5,039 peaks)
- name: CLIPper
  role: Source of previously reported ENCODE peak calls intersected via genomic interval
    overlap as the third caller in the ternary comparison (average ~6,904 peaks)
  repo: https://github.com/YeoLab/clipper
- name: Snakemake
  role: Workflow engine executing the Skipper pipeline (Snakemake 9.12.0 environment)
    that generates the reproducible_enriched_windows inputs
- name: STAR
  role: Aligner used to build the genome index (from GENCODE GFF and genome FASTA)
    for Skipper's eCLIP alignments
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:ab8271b1c7f0c1a5be4078cd957f56b9f36f314f3252dd78f8d0e1bea4bf3b06
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# ternary-and-scatter-method-comparison-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking visualization skill for comparing called binding-site counts across eCLIP peak/window callers: Skipper vs Piranha vs ENCODE CLIPper, rendered as Figure 2A ternary and Figure 2B scatter summaries with an aggregate method_comparison_counts.csv table.

## When to use

Use when you have per-RBP enriched-window/peak counts from two or more CLIP-seq callers (e.g., Skipper, Piranha, CLIPper) on the same ENCODE eCLIP datasets and must visualize and quantify how caller sensitivity compares.

## When NOT to use

- CLIP datasets lacking matched input controls — Skipper cannot construct its beta-binomial model, so the Skipper arm of the comparison cannot be computed
- Datasets where callers were run on different window tiling or genome builds — genomic interval overlap comparisons would be confounded and counts non-comparable
- Analyses not benchmarking caller sensitivity, e.g., motif discovery or fine-mapping of individual binding sites

## Inputs

- Matched IP and input eCLIP replicate FASTQs for benchmark RBPs from the ENCODE project portal
- Skipper reproducible_enriched_windows outputs (Snakemake pipeline, STAR-aligned with GENCODE GFF + genome FASTA index)
- Piranha peak calls computed on the same aligned replicates and identical tiled window framework
- Previously reported ENCODE CLIPper peak calls (BED-format coordinates)

## Outputs

- Figure 2A ternary plot of shared/unique binding-site fractions across Skipper, Piranha, and CLIPper
- Figure 2B scatter summary of pairwise per-RBP caller counts
- method_comparison_counts.csv with aggregated per-RBP peak counts for the three methods and the fraction of RBPs where Skipper calls more sites

## How to apply

Download matched IP and input eCLIP replicates for the benchmark RBPs from ENCODE, run the Skipper Snakemake pipeline (STAR alignment with a GENCODE-built index, tiled-window enriched-site calling) to obtain reproducible_enriched_windows, and run Piranha over the identical tiled window framework. Intersect Skipper and Piranha peak coordinates with previously reported ENCODE CLIPper peak calls via genomic interval overlap to assign shared and unique binding sites per RBP. Aggregate per-RBP counts for all three callers and compute the fraction of RBPs for which Skipper calls more sites than either existing method. Render a Figure 2A ternary plot of shared/unique fractions across the three callers and a Figure 2B scatter summary of pairwise counts, writing the aggregate table to method_comparison_counts.csv. Validate against the reported averages: Skipper ~21,310 windows, Piranha ~5,039 peaks, CLIPper ~6,904 peaks, with Skipper exceeding Piranha/CLIPper for ~72% of RBPs.

## Related tools

- **Skipper** (End-to-end eCLIP pipeline producing reproducible enriched windows; the caller whose sensitivity is benchmarked (average ~21,310 windows)) — https://github.com/YeoLab/skipper
- **Piranha** (Alternative peak caller run on the same aligned eCLIP replicates over the identical tiled window framework for the count comparison (average ~5,039 peaks))
- **CLIPper** (Source of previously reported ENCODE peak calls intersected via genomic interval overlap as the third caller in the ternary comparison (average ~6,904 peaks)) — https://github.com/YeoLab/clipper
- **Snakemake** (Workflow engine executing the Skipper pipeline (Snakemake 9.12.0 environment) that generates the reproducible_enriched_windows inputs)
- **STAR** (Aligner used to build the genome index (from GENCODE GFF and genome FASTA) for Skipper's eCLIP alignments)

## Evaluation signals

- Reproduce Skipper's average of ~21,310 reproducible enriched windows across the 219 ENCODE eCLIP datasets
- Reproduce Piranha's average of ~5,039 peaks and CLIPper's average of ~6,904 peaks on the same data
- Confirm Skipper detects more enriched windows than Piranha and CLIPper for ~72% of RBPs, matching the claim of 210%-320% more transcriptomic binding sites on average
- Ternary plot fractions for each RBP must sum to 1 across shared/unique caller assignments, and per-RBP interval-overlap assignments must be consistent between the ternary and scatter representations
- method_comparison_counts.csv must contain per-RBP rows with counts for all three callers plus the computed fraction where Skipper exceeds both alternatives

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model; CLIP datasets lacking these controls cannot be included in the comparison
- The comparison relies on genomic interval overlap against previously reported ENCODE CLIPper peaks, so discrepancies may reflect annotation/window-tiling choices rather than true biological differences
- Skipper sometimes calls >1,000% more sites than existing methods, so per-RBP scatter plots are dominated by large ratios and may need log scaling or careful axis choices

## Evidence

- [other] Task-card workflow: figure rendering and aggregate table: [evidence span withheld — anchor status: not_found]
