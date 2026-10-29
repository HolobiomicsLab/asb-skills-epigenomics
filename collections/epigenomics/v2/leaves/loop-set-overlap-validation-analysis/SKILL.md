---
name: loop-set-overlap-validation-analysis
description: Use when loops predicted from a non-Hi-C contact map (e.g., Micro-C H1-ESC
  or GM12878 DNA SPRITE coolers) must be benchmarked by their concordance with an
  independent Hi-C-derived loop set.
when_to_use_negative:
- The input contact map is itself Hi-C and already scored with platform-matched models
  — compare against pre-released predictions instead of revalidating cross-platform
- No orthogonal Hi-C loop reference set exists for the cell type, so overlap-based
  validation cannot be computed
- The goal is de novo loop detection on a platform with its own dedicated tool rather
  than cross-platform generalizability testing (e.g., DNA SPRITE lacks dedicated loop-detection
  algorithms, which is precisely why Peakachu transfer is used)
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Peakachu
  role: Trains CTCF ChIA-PET random-forest models per chromosome, scores cross-platform
    contact maps genome-wide with raw weights, and pools per-pixel probabilities into
    loop calls that are then overlapped with Hi-C loops
  repo: https://github.com/tariks/peakachu
- name: Juicer
  role: Hi-C processing platform whose .hic/.cool ecosystem and loop outputs (e.g.,
    HiCCUPS-style Hi-C loops via Juicebox tooling) provide the reference loop set
    used for overlap validation
  repo: https://github.com/aidenlab/juicer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:01b4e933b34c0ee221ba375e21c7673e4594017e4d3ce247aa3bc32048c0e76f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Juicer
  - Peakachu
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# loop-set-overlap-validation-analysis

## Summary

Validates chromatin loops predicted by Peakachu on non-Hi-C platforms (Micro-C, DNA SPRITE) by quantifying the fraction of predicted loops that overlap Hi-C-predicted loops, demonstrating that the trained models generalize across genome-wide contact-map technologies.

## When to use

Use when loops predicted from a non-Hi-C contact map (e.g., Micro-C H1-ESC or GM12878 DNA SPRITE coolers) must be benchmarked by their concordance with an independent Hi-C-derived loop set.

## When NOT to use

- The input contact map is itself Hi-C and already scored with platform-matched models — compare against pre-released predictions instead of revalidating cross-platform
- No orthogonal Hi-C loop reference set exists for the cell type, so overlap-based validation cannot be computed
- The goal is de novo loop detection on a platform with its own dedicated tool rather than cross-platform generalizability testing (e.g., DNA SPRITE lacks dedicated loop-detection algorithms, which is precisely why Peakachu transfer is used)

## Inputs

- Micro-C contact matrix (e.g., Krietenstein et al. 2020 H1-ESC) converted to cooler format at 5-10 kb bins
- DNA SPRITE contact matrix (e.g., Quinodoz et al. 2018 GM12878) converted to cooler format at 5-10 kb bins
- CTCF ChIA-PET positive interaction set in BEDPE format for training
- Reference Hi-C loop set (e.g., GM12878 Hi-C loops from Rao et al. 2014) for overlap validation

## Outputs

- Genome-wide per-pixel loop probability scores from score_genome
- Pooled, non-redundant loop call sets (Micro-C: 16,298 loops in H1-ESC; DNA SPRITE: 1,731 loops in GM12878)
- Overlap/validation statistics (fraction of cross-platform loops overlapping Hi-C-predicted loops: 74% Micro-C, 77% DNA SPRITE)

## How to apply

Convert the Micro-C (Krietenstein et al. 2020, H1-ESC) or DNA SPRITE (Quinodoz et al. 2018, GM12878) contact matrices to cooler format at 5-10 kb bin resolution, then train Peakachu random-forest models on the CTCF ChIA-PET positive interaction set with 'peakachu train' at 10 kb resolution, using chromosome-wise splitting (each chromosome's model is trained on the other 22 chromosomes) to avoid overfitting. Score the raw (not ICE-normalized) contact values with 'peakachu score_genome --clr-weight-name raw', since Micro-C and DNA SPRITE use raw rather than weighted contacts, then pool per-pixel probabilities with 'peakachu pool' (default --minimum-prob 0.5; tune the -t cutoff around 0.9-0.95 for the sensitivity-specificity tradeoff). Finally, intersect the pooled loop calls with Hi-C-predicted loops and report the overlap rate. Correct application reproduces the published numbers: 16,298 Micro-C loops in H1-ESC with 74% Hi-C overlap, and 1,731 DNA SPRITE loops in GM12878 with 77% validated by Hi-C-predicted loops.

## Related tools

- **Peakachu** (Trains CTCF ChIA-PET random-forest models per chromosome, scores cross-platform contact maps genome-wide with raw weights, and pools per-pixel probabilities into loop calls that are then overlapped with Hi-C loops) — https://github.com/tariks/peakachu
- **Juicer** (Hi-C processing platform whose .hic/.cool ecosystem and loop outputs (e.g., HiCCUPS-style Hi-C loops via Juicebox tooling) provide the reference loop set used for overlap validation) — https://github.com/aidenlab/juicer

## Examples

```
peakachu score_genome -r 10000 -p h1esc.microc.10kb.cool --clr-weight-name raw -m models/ -O microc_scores.tsv
```

## Evaluation signals

- Pooled Micro-C loop count in H1-ESC is on the order of the published 16,298 loops (74%, i.e., 8315/11,247, overlapping Hi-C loops)
- Pooled DNA SPRITE loop count in GM12878 is near the published 1,731 loops, with ~77% validated by Hi-C-predicted loops
- score_genome was run with --clr-weight-name raw (not ICE weights) on Micro-C and DNA SPRITE coolers
- Loop probability cutoffs fall in the 0.9-0.95 range (paper used 0.97 for the CTCF model in GM12878; default --minimum-prob is 0.5 for per-pixel output)
- Trained model set contains 23 per-chromosome pickled models, each trained on interactions from the other 22 chromosomes

## Limitations

- DNA SPRITE and similar platforms lack dedicated loop-detection algorithms, so cross-platform calls depend entirely on model transfer from Hi-C/ChIA-PET training data
- Each method (Peakachu, HiCCUPS, Fit-Hi-C) detects a unique set of chromatin loops, so overlap rates below 100% are expected and the complete loop set may not be captured by any single method
- Tuning of the pooling probability cutoff (-t around 0.9-0.95) is required to balance sensitivity and specificity; cutoffs used in other cell lines are only reported in Supplementary Data 2

## Evidence

- [intro] Platform-agnostic applicability on Micro-C and DNA SPRITE: "Finally, we show Peakachu is a platform-agnostic tool by applying it in two additional genome-wide interaction data types, Micro-C, and DNA SPRITE."
- [discussion] Generalizability demonstrated on DNA SPRITE: "we show the generalizability of our framework by demonstrating its performance in a DNA SPRITE data matrix"
- [readme] Chromosome-wise model training to avoid overfitting: "This will train 23 random forest models, each labeled by a chromosome. The model for every chromosome was trained using interactions from all the other 22 chromosomes"
- [readme] Raw signal weighting for non-Hi-C platforms: "The name of the weight column in your Cooler URI for normalizing the contact signals. Specify it to "raw" if you want to use the raw signals."
- [discussion] CTCF model probability cutoff: "we set the cutoff to 0.97 and 0.92 for the CTCF model and the H3K27ac model, respectively."
