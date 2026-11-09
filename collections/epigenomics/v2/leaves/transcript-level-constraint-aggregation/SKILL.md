---
name: transcript-level-constraint-aggregation
description: Use when per-window variant-depletion delta scores exist for RBP binding
  windows and you need transcript-level nomination of constrained RBP-transcript pairs
  with FDR control.
when_to_use_negative:
- CLIP datasets lacking a matched input sample, since Skipper cannot construct its
  beta-binomial model and no fine-mapped windows exist to aggregate
- You need binding-site-level (window-level) constraint calls rather than transcript-level
  hypotheses; per-window constraint signal is noisier and the transcript-level step
  is explicitly an aggregation
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: Skipper
  role: Produces the fine-mapped, reproducible enriched windows per transcript whose
    constraint signal is aggregated to the transcript level
  repo: https://github.com/YeoLab/skipper
- name: gapped k-mer SVM
  role: LS-GKM models provide the per-variant delta scores that are regressed against
    gnomAD allele-frequency bins and aggregated
  repo: https://github.com/Dongwon-Lee/lsgkm
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:30e8dee77dabe274ab91ddb9cebd477bbb3b94938425502f5eea81d0db2d84bf
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
  - gapped k-mer SVM
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# transcript-level-constraint-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-window selective-constraint evidence (mean LS-GKM delta score regressed against gnomAD allele-frequency bins) up to the RBP-transcript level and applies multiple-testing correction at 10% FDR to nominate constrained translation factor binding targets.

## When to use

Use when per-window variant-depletion delta scores exist for RBP binding windows and you need transcript-level nomination of constrained RBP-transcript pairs with FDR control.

## When NOT to use

- CLIP datasets lacking a matched input sample, since Skipper cannot construct its beta-binomial model and no fine-mapped windows exist to aggregate
- You need binding-site-level (window-level) constraint calls rather than transcript-level hypotheses; per-window constraint signal is noisier and the transcript-level step is explicitly an aggregation

## Inputs

- Per-window gnomAD SNV delta scores from trained LS-GKM gapped k-mer SVM models (reference vs alternate allele predictions)
- gnomAD allele-frequency bins per RBP
- Fine-mapped, reproducible enriched windows from Skipper output (75-nt windows on GENCODE GRCh38 transcripts)
- RBP-to-transcript mapping from Skipper annotated binding sites

## Outputs

- constrained_rbp_transcript_pairs.tsv — nominated constrained RBP-transcript pairs at 10% FDR
- Expected nomination set of 65 constrained RBP-transcript pairs (reproducing Figure 7C / Table S9)

## How to apply

After LS-GKM delta scores are computed for gnomAD SNVs within 75-nt fine-mapped Skipper enriched windows and binned by gnomAD allele frequency, aggregate the constraint signal from windows to the transcript level for each RBP. Transcript-level aggregation is chosen because it is a 'well-powered intermediate approach' between single-window tests (underpowered) and genome-wide aggregation (too coarse). Apply multiple-testing correction across all RBP-transcript pairs and call a pair constrained at a 10% FDR threshold. Write the nominated set to constrained_rbp_transcript_pairs.tsv and benchmark it against the 65 constrained pairs reported in Figure 7C / Table S9 of the paper. Decision point: if transcript counts diverge from 65, revisit window filtering (GINI_CUTOFF, THRESHOLD_MIN) and the allele-frequency binning before re-aggregating.

## Related tools

- **Skipper** (Produces the fine-mapped, reproducible enriched windows per transcript whose constraint signal is aggregated to the transcript level) — https://github.com/YeoLab/skipper
- **gapped k-mer SVM** (LS-GKM models provide the per-variant delta scores that are regressed against gnomAD allele-frequency bins and aggregated) — https://github.com/Dongwon-Lee/lsgkm

## Examples

```
gkmtrain bound.fa unbound.fa rbp_model && gkmpredict windows.fa rbp_model.model.txt scores.txt; Rscript aggregate_constraint.R --deltas scores.txt --af-bins gnomad_af_bins.tsv --fdr 0.10 -o constrained_rbp_transcript_pairs.tsv
```

## Evaluation signals

- The nominated set at 10% FDR reproduces the 65 constrained RBP-transcript pairs reported in Figure 7C / Table S9 of the paper
- constrained_rbp_transcript_pairs.tsv is written with one row per RBP-transcript pair and a corrected FDR value ≤ 0.10 for every nominated pair
- Every nominated transcript has at least one fine-mapped Skipper enriched window with scored gnomAD SNVs spanning multiple allele-frequency bins
- Constraint direction is consistent: mean delta score regressed against allele-frequency bin shows depletion of disruptive variants in occupied windows
- Aggregated signals derive only from windows passing Skipper's GFF filtration and enriched-window identification steps

## Limitations

- Transcript-level aggregation is a hypothesis-generating intermediate; it identifies constrained transcripts but does not pinpoint which individual binding window or variant drives the signal
- The gapped k-mer SVM used to model binding-site occupancy is less sophisticated than emerging deep learning-based approaches
- Best practices for blacklisting reproducible regions are not established, and blacklist choices can affect which windows feed the aggregation

## Evidence
