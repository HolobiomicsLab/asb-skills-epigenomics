---
name: sequencing-depth-matched-model-selection
description: Use when scoring/predicting chromatin loops on a new Hi-C, Micro-C, or
  DNA SPRITE contact map (.cool/.hic) in a cell type or species different from the
  training set, and a depth-matched pre-trained model must be chosen.
when_to_use_negative:
- The contact map has fewer than ~30 million intrachromosomal reads, where loop detection
  at high resolution is unreliable even with the depth-robust models
- You are training a new model from scratch rather than applying an existing pre-trained
  model — use the train workflow instead
tools:
- name: Peakachu
  role: Provides the 'depth' subcommand that counts intra-chromosomal contacts and
    selects the most appropriate pre-trained Random Forest loop-prediction model;
    its 'score_genome' and 'pool' subcommands are then run with the selected model
  repo: https://github.com/tariks/peakachu
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:9ffcbabf96cb4f53586c9118f66728fbd93b72296e2b9cb161ad601e4e822f7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Peakachu
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Sequencing-depth-matched model selection

## Summary

Selects the appropriate pre-trained Peakachu Random Forest loop-prediction model by first measuring the total number of intra-chromosomal contacts in a Hi-C contact map, since Peakachu models are robust to sequencing depth only when matched to it (usable down to ~30 million intrachromosomal reads).

## When to use

Use when scoring/predicting chromatin loops on a new Hi-C, Micro-C, or DNA SPRITE contact map (.cool/.hic) in a cell type or species different from the training set, and a depth-matched pre-trained model must be chosen.

## When NOT to use

- The contact map has fewer than ~30 million intrachromosomal reads, where loop detection at high resolution is unreliable even with the depth-robust models
- You are training a new model from scratch rather than applying an existing pre-trained model — use the train workflow instead

## Inputs

- ICE-normalized .cool or .hic contact map (e.g. 10 kb resolution, --clr-weight-name weight)
- Pre-trained Peakachu Random Forest models (e.g. GM12878 CTCF ChIA-PET-trained, K562-trained, mESC SMC1 HiChIP-trained)

## Outputs

- Total intra-chromosomal contact count and recommended model for the input map
- Per-pixel loop probability scores from score_genome (after depth-matched model selection)
- Non-redundant loop calls (bedpe) from pool at the model-specific probability cutoff

## How to apply

First run the Peakachu 'depth' subcommand on the ICE-normalized cooler (e.g. --clr-weight-name weight at 10 kb resolution) to compute the total intra-chromosomal read count and have the tool recommend the most appropriate pre-trained model for that depth. Only proceed if the map has at least ~30 million intrachromosomal reads, the minimum depth at which Peakachu reliably detects high-resolution loops; down-sampled GM12878 maps were generated with binomial probability without re-mapping to build the depth-robustness benchmarks. Once the model is selected (e.g. the GM12878 CTCF ChIA-PET-trained model for cross-cell-type transfer to K562 or mESC), run 'score_genome' at 10 kb and then 'pool' with the model-specific cutoff (0.97 for the CTCF model, 0.92 for the H3K27ac model) to extract non-redundant loop calls. The rationale is that model transferability across cell types and species holds when sequencing depth is matched, as verified by the 78% (K562) and 69%/74% (mESC) overlap between transferred and matched-cell-type model predictions.

## Related tools

- **Peakachu** (Provides the 'depth' subcommand that counts intra-chromosomal contacts and selects the most appropriate pre-trained Random Forest loop-prediction model; its 'score_genome' and 'pool' subcommands are then run with the selected model) — https://github.com/tariks/peakachu

## Examples

```
peakachu depth -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool
```

## Evaluation signals

- The 'depth' output reports a total intra-chromosomal read count consistent with the map's resolution and coverage (e.g. at least ~30 million intrachromosomal reads)
- The model recommended by 'depth' matches the sequencing depth tier used for subsequent score_genome runs
- Down-sampling checks (binomial down-sampling without re-mapping) reproduce loop recovery consistent with the paper's depth-robustness benchmark
- Cross-cell-type transfer with the selected model recovers overlap comparable to published figures (78% K562; 69%/74% mESC) between transferred and matched-cell-type model loop calls

## Limitations

- Probability cutoffs for cell lines other than GM12878 (0.97 CTCF / 0.92 H3K27ac) are only available in Supplementary Data 2, not in the main text
- Robustness to sequencing depth extends down to ~30 million intrachromosomal reads; maps below this depth are not supported by the demonstrated workflow
- Different model tiers must be selected per depth; using a mismatched depth model can degrade loop predictions

## Evidence

- [readme] depth command purpose: "depth Calculate the total number of intra-chromosomal chromatin contacts and select the most appropriate pre-trained model for you."
- [discussion] depth robustness finding: "The major improvement of Peakachu is its robustness to sequencing depth, which makes it applicable for predicting chromatin loops in Hi-C data with only ~30 million intrachromosomal reads."
- [abstract] cross-platform/species transfer: "We show that our models perform well in different platforms, across different sequencing depths, and across different species."
- [discussion] down-sampling procedure: "The contact maps were down-sampled using a binomial probability without re-mapping."
