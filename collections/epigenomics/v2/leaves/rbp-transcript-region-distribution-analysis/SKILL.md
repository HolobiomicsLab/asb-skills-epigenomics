---
name: rbp-transcript-region-distribution-analysis
description: Use when eCLIP replicates (e.g., WT vs point-mutant RBP) have Skipper
  reproducible_enriched_windows output and the goal is to compare the fraction of
  binding windows assigned to transcript regions such as CDS, deep introns, or tRNA
  loci.
when_to_use_negative:
- CLIP datasets lacking a matched input sample, since Skipper requires matched input
  controls to construct the beta-binomial model for statistical testing
- Analyses where raw peak-caller output (e.g., CLIPper BED peaks), not Skipper reproducible_enriched_windows,
  is the only available binding-site representation
- Questions about the mechanism of binding change (complex formation vs localization
  vs binding preference), which eCLIP data alone cannot resolve
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0659
tools:
- name: Skipper
  role: End-to-end eCLIP pipeline producing reproducible_enriched_windows with transcript-region
    annotations (feature_type_top) used for region proportion aggregation
  repo: https://github.com/YeoLab/skipper
- name: STAR
  role: Aligns trimmed eCLIP reads during Skipper's alignment step using the pre-built
    genome index
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:cdd49bd33ef8751a6c74c6c9dc735321edae5f7b2e819bb602ad9efdf09f53c2
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
  - STAR
  - Skipper
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# rbp-transcript-region-distribution-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how an RBP's reproducible enriched CLIP-seq windows distribute across annotated transcript regions (CDS, UTRs, introns, repeats) using Skipper's fixed-window annotation framework, enabling detection of WT-vs-mutant binding profile shifts such as the RPS19 R62W/R101H CDS-increase and deep-intron-decrease signature.

## When to use

Use when eCLIP replicates (e.g., WT vs point-mutant RBP) have Skipper reproducible_enriched_windows output and the goal is to compare the fraction of binding windows assigned to transcript regions such as CDS, deep introns, or tRNA loci.

## When NOT to use

- CLIP datasets lacking a matched input sample, since Skipper requires matched input controls to construct the beta-binomial model for statistical testing
- Analyses where raw peak-caller output (e.g., CLIPper BED peaks), not Skipper reproducible_enriched_windows, is the only available binding-site representation
- Questions about the mechanism of binding change (complex formation vs localization vs binding preference), which eCLIP data alone cannot resolve

## Inputs

- Skipper reproducible_enriched_windows tables per eCLIP replicate (WT, R62W, R101H)
- eCLIP FASTQ files with matched input/control samples per replicate
- Filtered GENCODE or Ensembl GFF3 (e.g., gencode.v49.basic.annotation.gff3) with one best transcript per gene
- GRCh38 primary assembly genome FASTA for the STAR index

## Outputs

- Per-sample feature proportion table (fraction of reproducible enriched windows per feature_type_top region)
- WT-vs-mutant region proportion comparison figure (e.g., figure7e_feature_proportions.pdf)
- Per-replicate tRNA (or other feature-class) enrichment values with paired t-test p-values and annotated comparison figure (e.g., figure7f_trna_enrichment.pdf)

## How to apply

Run Skipper end-to-end (preprocessing/trimming, STAR alignment, GFF partitioning, enriched window identification, fine-mapping) on each eCLIP replicate with its matched input control, using a pre-filtered GENCODE/Ensembl GFF3 where one best transcript per gene is selected. Load the resulting reproducible_enriched_windows tables per sample and aggregate windows by the feature_type_top annotation to compute the per-sample fraction of windows in each transcript region. For mutant comparisons, quantify the WT-to-mutant shift in region proportions (e.g., CDS 33%->45%, deep intronic 20%->7%) and render the comparison as a proportions figure; for specific feature classes such as tRNA loci, subset windows to that feature, compute per-replicate enrichment, and apply paired t-tests across matched WT/mutant replicates. Rationale: Skipper skips peak calling by tiling fixed windows over annotated transcripts, so region-level aggregation is directly supported by the annotation columns and avoids peak-caller biases against intronic or low-abundance signal.

## Related tools

- **Skipper** (End-to-end eCLIP pipeline producing reproducible_enriched_windows with transcript-region annotations (feature_type_top) used for region proportion aggregation) — https://github.com/YeoLab/skipper
- **STAR** (Aligns trimmed eCLIP reads during Skipper's alignment step using the pre-built genome index)

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Computed CDS window fraction for mutant RPS19 rises from 33% (WT) to 45% (R62W/R101H)
- Computed deep intronic window fraction falls from 20% (WT) to 7% in mutants, with region-proportion comparison significant at p < 2.2e-16
- Paired t-tests across matched WT/mutant replicates yield significant tRNA enrichment p-values for both mutants
- Aggregated feature fractions sum to ~1 across feature_type_top categories within each sample
- Figures (e.g., figure7e_feature_proportions.pdf, figure7f_trna_enrichment.pdf) render with the expected direction of shift for every matched replicate pair

## Limitations

- eCLIP data alone cannot determine whether observed binding changes arise from differences in complex formation, localization, binding preference, or another mechanism
- Expressing variant ORFs (e.g., RPS19 mutants) in HEK293T cells may not reflect the true disease state
- Skipper requires a matched input sample to construct the beta-binomial model; datasets without controls cannot be analyzed
- Peak-calling-style analyses risk false negatives at exon-intron boundaries where intronic signal is masked by exonic reads, so results are not comparable if derived from peak callers instead of Skipper windows

## Evidence
