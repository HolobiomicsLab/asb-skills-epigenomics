---
name: multiple-testing-fdr-correction
description: Use when you have per-RBP constraint regressions (mean delta score vs.
  gnomAD allele-frequency bin) over Skipper enriched windows and must aggregate to
  RBP-transcript pairs and control the false discovery rate to nominate constrained
  transcripts.
when_to_use_negative:
- Your dataset lacks a matched input control sample, so Skipper cannot build the beta-binomial
  model and no reproducible enriched windows exist to aggregate for constraint testing
- You need window-level or site-level constraint calls; the paper's framework nominates
  only RBP-transcript pairs after transcript-level aggregation, not individual constrained
  binding windows
- You are testing within a single pre-specified RBP-transcript hypothesis, where a
  single statistical test needs no FDR correction
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: Skipper
  role: Produces the fine-mapped, reproducible enriched binding windows whose constraint
    signal is aggregated to the transcript level before FDR correction
  repo: https://github.com/YeoLab/skipper.git
- name: gapped k-mer SVM
  role: LS-GKM models generate the variant delta scores regressed against gnomAD allele-frequency
    bins; the resulting constraint statistics are the input to transcript-level FDR
    correction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:5ac2eddf92b711c858b1c9b355f5de962fced9f8ee1283a2a28fcfe2025351f3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  repo_url: https://github.com/YeoLab/skipper.git
  tools:
  - Skipper
  - gapped k-mer SVM
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multiple-testing-fdr-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates selective-constraint signal (gnomAD allele-frequency-bin regression of LS-GKM delta scores) from Skipper-defined translation factor binding windows up to the transcript level, then applies multiple-testing correction at a 10% FDR threshold to nominate constrained RBP-transcript pairs. Used because per-window constraint tests are underpowered individually, while transcript-level aggregation is a well-powered hypothesis-generating intermediate.

## When to use

Use when you have per-RBP constraint regressions (mean delta score vs. gnomAD allele-frequency bin) over Skipper enriched windows and must aggregate to RBP-transcript pairs and control the false discovery rate to nominate constrained transcripts.

## When NOT to use

- Your dataset lacks a matched input control sample, so Skipper cannot build the beta-binomial model and no reproducible enriched windows exist to aggregate for constraint testing
- You need window-level or site-level constraint calls; the paper's framework nominates only RBP-transcript pairs after transcript-level aggregation, not individual constrained binding windows
- You are testing within a single pre-specified RBP-transcript hypothesis, where a single statistical test needs no FDR correction

## Inputs

- Per-variant LS-GKM delta scores for gnomAD SNVs overlapping Skipper fine-mapped enriched windows (75-nt, GENCODE GRCh38 centered)
- gnomAD allele frequency annotations for each scored variant
- Per-RBP transcript annotations mapping enriched windows to GENCODE transcripts
- Regression statistics (mean delta score vs. allele-frequency bin) per RBP

## Outputs

- constrained_rbp_transcript_pairs.tsv: table of constrained RBP-transcript pairs passing 10% FDR
- Transcript-level aggregated constraint statistics with corrected q-values per RBP-transcript pair

## How to apply

For each RBP, score gnomAD SNVs overlapping Skipper fine-mapped enriched windows with the trained LS-GKM model, computing a delta score per variant as the difference between reference- and alternate-allele predictions. Bin variants by gnomAD allele frequency and regress mean delta score against allele-frequency bin to obtain a per-RBP constraint statistic. Aggregate the constraint signal to the transcript level for each RBP — the paper's rationale is that transcript-level aggregation is a 'well-powered intermediate approach' for hypothesis generation. Apply multiple-testing correction across the RBP-transcript tests and threshold at 10% FDR to nominate constrained pairs. Write the final results to constrained_rbp_transcript_pairs.tsv and verify the nominated set against the published result.

## Related tools

- **Skipper** (Produces the fine-mapped, reproducible enriched binding windows whose constraint signal is aggregated to the transcript level before FDR correction) — https://github.com/YeoLab/skipper.git
- **gapped k-mer SVM** (LS-GKM models generate the variant delta scores regressed against gnomAD allele-frequency bins; the resulting constraint statistics are the input to transcript-level FDR correction)

## Evaluation signals

- The nominated set at 10% FDR reproduces the 65 constrained RBP-transcript pairs reported in Figure 7C / Table S9 of the paper
- Every pair in constrained_rbp_transcript_pairs.tsv has a corrected q-value ≤ 0.10
- Constrained transcripts show the expected directionality: mean delta score decreases across gnomAD allele-frequency bins (depletion of disruptive variants in occupied sites)
- Each nominated pair maps to a transcript passing the GFF best-transcript-per-gene filter used by Skipper
- Aggregation is done at the transcript level per RBP — spot-check that window-level signals from a single transcript are collapsed before correction

## Limitations

- Transcript-level aggregation is a hypothesis-generating intermediate, not a definitive functional assignment; nominated transcripts require downstream validation
- Post hoc fine-mapping does not learn sequence motifs or binding affinity directly from CLIP data, so constraint calls depend on the LS-GKM delta-score approximation of variant effects
- eCLIP data alone do not determine the mechanism underlying observed binding changes, which limits interpretation of nominated pairs

## Evidence

- [methods] Workflow definition of transcript-level aggregation and 10% FDR: [evidence span withheld — anchor status: not_found]
