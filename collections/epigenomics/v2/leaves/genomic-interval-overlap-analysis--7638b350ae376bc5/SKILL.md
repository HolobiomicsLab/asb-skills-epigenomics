---
name: genomic-interval-overlap-analysis
description: Use when comparing peak/enriched-window calls from multiple CLIP-seq
  analysis methods run on the same matched IP-and-input eCLIP replicates, and you
  need per-RBP counts of shared vs. unique binding sites via interval intersection.
when_to_use_negative:
- The CLIP-seq dataset lacks a matched input sample — Skipper requires matched input
  to construct its beta-binomial model, so the Skipper arm of the comparison cannot
  be run
- You only need peak calling for a single method on a single dataset rather than a
  cross-caller benchmark comparison of interval overlaps
- The calls are already reduced to per-RBP aggregate counts — no interval overlap
  is needed; proceed directly to count aggregation and plotting
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3308
tools:
- name: Skipper
  role: End-to-end Snakemake eCLIP pipeline producing reproducible enriched windows
    (the caller being benchmarked)
  repo: https://github.com/YeoLab/skipper
- name: Piranha
  role: Alternative peak caller run on the same aligned eCLIP replicates over the
    identical tiled window framework
- name: CLIPper
  role: Existing peak caller whose previously reported ENCODE peak calls (BED output)
    provide the reference calls for interval overlap
  repo: https://github.com/YeoLab/clipper
- name: STAR
  role: Aligner whose genome index (built from the GENCODE GFF3 and genome FASTA)
    supplies the aligned replicates fed to all callers
- name: Snakemake
  role: Workflow engine (version 9.12.0) orchestrating the Skipper preprocessing,
    alignment, and enriched-window identification steps
  repo: https://github.com/YeoLab/skipper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:0fc3624a3af7f098ed1a3328ba1c86f671553e0115df10f420f39c76af273451
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# genomic-interval-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks CLIP-seq enriched-site callers by intersecting peak/window coordinates (e.g., Skipper vs. Piranha vs. ENCODE CLIPper calls) across the same eCLIP replicates using genomic interval overlap, then aggregating per-RBP counts to quantify how many more transcriptomic binding sites one caller detects.

## When to use

Use when comparing peak/enriched-window calls from multiple CLIP-seq analysis methods run on the same matched IP-and-input eCLIP replicates, and you need per-RBP counts of shared vs. unique binding sites via interval intersection.

## When NOT to use

- The CLIP-seq dataset lacks a matched input sample — Skipper requires matched input to construct its beta-binomial model, so the Skipper arm of the comparison cannot be run
- You only need peak calling for a single method on a single dataset rather than a cross-caller benchmark comparison of interval overlaps
- The calls are already reduced to per-RBP aggregate counts — no interval overlap is needed; proceed directly to count aggregation and plotting

## Inputs

- Matched IP and input eCLIP replicates for benchmark RBPs from the ENCODE project portal
- Skipper reproducible_enriched_windows outputs from the Snakemake pipeline
- Piranha peak calls computed over the same tiled window framework on identical aligned replicates
- Previously reported ENCODE CLIPper peak calls (BED files from 'clipper -b BAM -o OUT.bed -s hg19')
- STAR genome index built from a GENCODE/Ensembl GFF3 (e.g., gencode.v49.basic.annotation.gff3) and genome FASTA (e.g., GRCh38.primary_assembly.genome.fa)

## Outputs

- Shared and unique binding-site assignments per RBP from genomic interval overlap of Skipper, Piranha, and CLIPper coordinates
- Aggregate per-RBP peak counts per caller (Skipper, Piranha, CLIPper), written to method_comparison_counts.csv
- Figure 2A ternary plot and Figure 2B scatter summaries of the caller count comparison
- Fraction of RBPs for which Skipper calls more sites than either existing method (~72%)

## How to apply

Download matched IP and input eCLIP replicates for the benchmark RBPs from the ENCODE portal. Run the Skipper end-to-end Snakemake pipeline (preprocessing/trimming, alignment with a STAR genome index built from a GENCODE GFF3 such as gencode.v49.basic.annotation.gff3 and the GRCh38.primary_assembly.genome.fa, then window-based enriched-site calling) to obtain reproducible_enriched_windows outputs. Run Piranha on the same aligned replicates over the identical tiled window framework, and take the previously reported ENCODE CLIPper peak calls (BED format, e.g., produced by 'clipper -b ... -o OUT.bed -s hg19' followed by input normalization via the separate merge_peaks repository). Intersect Skipper and Piranha coordinates with the CLIPper BEDs using genomic interval overlap to assign shared and unique binding sites per RBP, then aggregate per-caller peak counts and compute the fraction of RBPs for which Skipper calls more sites. Validation targets: Skipper averages ~21,310 reproducible enriched windows across 219 ENCODE eCLIP datasets versus ~5,039 for Piranha and ~6,904 for CLIPper, with Skipper exceeding the existing callers for ~72% of RBPs, consistent with the reported 210%-320% more transcriptomic binding sites.

## Related tools

- **Skipper** (End-to-end Snakemake eCLIP pipeline producing reproducible enriched windows (the caller being benchmarked)) — https://github.com/YeoLab/skipper
- **Piranha** (Alternative peak caller run on the same aligned eCLIP replicates over the identical tiled window framework)
- **CLIPper** (Existing peak caller whose previously reported ENCODE peak calls (BED output) provide the reference calls for interval overlap) — https://github.com/YeoLab/clipper
- **STAR** (Aligner whose genome index (built from the GENCODE GFF3 and genome FASTA) supplies the aligned replicates fed to all callers)
- **Snakemake** (Workflow engine (version 9.12.0) orchestrating the Skipper preprocessing, alignment, and enriched-window identification steps) — https://github.com/YeoLab/skipper

## Examples

```
clipper -b YOUR_BAM_FILE.bam -o YOUR_OUT_FILE.bed -s hg19
```

## Evaluation signals

- Skipper average reproducible enriched windows across 219 ENCODE eCLIP datasets reproduces to ~21,310 (vs. ~5,039 for Piranha and ~6,904 for CLIPper)
- For ~72% of RBPs, Skipper's interval-overlap-derived count exceeds both Piranha and CLIPper counts
- Ratio of Skipper to existing-caller sites is consistent with the reported 210%-320% more transcriptomic binding sites (sometimes >1,000%)
- method_comparison_counts.csv contains per-RBP counts for all three callers with shared/unique assignments resolvable to overlapping intervals
- Figure 2A ternary and Figure 2B scatter render from the aggregated counts without missing per-RBP entries

## Limitations

- Skipper requires a matched input sample to construct the beta-binomial model for statistical testing: CLIP datasets lacking these controls currently cannot be analyzed using Skipper, so they cannot enter the comparison
- CLIPper does not include input normalization itself; the input normalization pipeline lives in the separate merge_peaks repository and must be applied before calls are comparable
- Peak-calling approaches are susceptible to false negatives (e.g., masking of intronic signal by exonic reads at exon-intron boundaries), which can bias cross-caller count comparisons

## Evidence

- [results] Skipper vs existing caller averages: [evidence span withheld — anchor status: not_found]
- [readme] CLIPper CLI for generating BED peak calls: [evidence span withheld — anchor status: external_doc]
- [readme] CLIPper input normalization caveat: [evidence span withheld — anchor status: external_doc]
