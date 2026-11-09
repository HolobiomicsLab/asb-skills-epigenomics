---
name: pretrained-model-transfer-across-cell-types
description: Use when you need to predict chromatin loops in a Hi-C contact map without
  training a new model, by transferring an existing Peakachu model trained in a different
  cell type or species, and quantifying overlap with cell-type-matched models.
when_to_use_negative:
- You have cell-type-matched ChIA-PET/HiChIP positive sets and sufficient data to
  train a dedicated model for the target cell type — training on matched data is preferable
  to transfer.
- The target contact map is not a genome-wide Hi-C-style interaction matrix (e.g.,
  you need de novo loop detection on raw paired-end data rather than scoring a binned
  cooler map).
- Sequencing depth is far below the ~30 million intra-chromosomal read regime the
  models were validated for, so depth-appropriate model selection via 'peakachu depth'
  is not possible.
edam_operation: http://edamontology.org/operation_3903
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_3474
tools:
- name: Peakachu
  role: Random Forest framework used for depth assessment (depth), genome-wide per-pixel
    loop probability scoring (score_genome), loop pooling with probability cutoff
    (pool), and transfer of pre-trained models across cell types and species
  repo: https://github.com/tariks/peakachu
- name: Juicer
  role: Hi-C processing platform whose .hic format and .cool-compatible outputs serve
    as the contact-map input that Peakachu (version>=1.1.2) supports
  repo: https://github.com/aidenlab/juicer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:7452c3ded65d05622b87931ebb04f889d886ad56fc54b907575fd714204e25fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Juicer
  - Peakachu
  edam_operation: http://edamontology.org/operation_3903
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# pretrained-model-transfer-across-cell-types

## Summary

Apply a Peakachu Random Forest model pre-trained on one cell type's ChIA-PET/HiChIP loops (e.g., GM12878 CTCF ChIA-PET) to score and call loops in Hi-C maps from other cell types and species (K562, mESC), testing cross-cell-type and cross-species transferability of the supervised loop detector.

## When to use

Use when you need to predict chromatin loops in a Hi-C contact map without training a new model, by transferring an existing Peakachu model trained in a different cell type or species, and quantifying overlap with cell-type-matched models.

## When NOT to use

- You have cell-type-matched ChIA-PET/HiChIP positive sets and sufficient data to train a dedicated model for the target cell type — training on matched data is preferable to transfer.
- The target contact map is not a genome-wide Hi-C-style interaction matrix (e.g., you need de novo loop detection on raw paired-end data rather than scoring a binned cooler map).
- Sequencing depth is far below the ~30 million intra-chromosomal read regime the models were validated for, so depth-appropriate model selection via 'peakachu depth' is not possible.

## Inputs

- ICE-normalized Hi-C contact maps (.cool or .hic) at 10 kb resolution for target cell types/species (K562, mESC)
- Pre-trained Peakachu Random Forest model pickles (e.g., GM12878 CTCF ChIA-PET-trained, K562-trained, mESC SMC1 HiChIP-trained)
- Cell-type-matched probability cutoffs (e.g., 0.97 for the CTCF model, from Supplementary Data 2)

## Outputs

- Per-pixel loop probability scores genome-wide from score_genome
- Non-redundant loop calls in bedpe format from pool (targeting ~13,566 loops in K562 and ~14,842 in mESC)
- Cross-model overlap table reporting fraction of transferred-model loops recovered by matched cell-type models (78% K562; 69%/74% mESC)

## How to apply

First run 'peakachu depth' on the target K562 and mESC Hi-C matrices to count total intra-chromosomal reads and confirm selection of the sequencing-depth-matched pre-trained model. Then run 'peakachu score_genome' at 10 kb resolution on each ICE-normalized contact map (--clr-weight-name weight) using the transferred GM12878 CTCF ChIA-PET-trained model to compute per-pixel loop probabilities. Apply 'peakachu pool' with the CTCF-model probability cutoff of 0.97 to extract non-redundant loop calls, expecting roughly 13,566 loops in K562 and 14,842 loops in mESC. In parallel, run score_genome and pool with the cell-type-matched models (K562-trained and mESC SMC1 HiChIP-trained) on the same maps. Finally, compute the fraction of transferred-model predictions recovered by each matched model, targeting the reported overlaps of 78% for K562 and 69%/74% for mESC.

## Related tools

- **Peakachu** (Random Forest framework used for depth assessment (depth), genome-wide per-pixel loop probability scoring (score_genome), loop pooling with probability cutoff (pool), and transfer of pre-trained models across cell types and species) — https://github.com/tariks/peakachu
- **Juicer** (Hi-C processing platform whose .hic format and .cool-compatible outputs serve as the contact-map input that Peakachu (version>=1.1.2) supports) — https://github.com/aidenlab/juicer

## Examples

```
peakachu score_genome -r 10000 -p k562.10kb.cool --clr-weight-name weight -m gm12878_ctcf_model.pkl -O k562_scores.txt && peakachu pool -r 10000 -i k562_scores.txt --minimum-prob 0.97 -O k562_loops.bedpe
```

## Evaluation signals

- Loop bedpe files contain approximately the reported loop counts: 13,566 loops in K562 and 14,842 loops in mESC (within reasonable tolerance).
- Overlap analysis reproduces the reported transferability figures: 78% (10,571/13,566) of GM12878-model K562 predictions matched by the K562-trained model, and 69%/74% overlap in mESC.
- 'peakachu depth' output confirms sequencing-depth-matched model selection for each target matrix (total intra-chromosomal read counts).
- score_genome was run with ICE normalization weights (--clr-weight-name weight) at 10 kb resolution, and pool applied the CTCF-model cutoff of 0.97.

## Limitations

- Probability cutoffs for non-GM12878 cell lines are only documented in Supplementary Data 2, not the main text, so exact cutoffs for new cell types may need to be obtained or estimated.
- A large proportion of predictions are shared across methods, but Peakachu, HiCCUPS, and Fit-Hi-C each detect a unique set of loops, so transferred-model loops will not capture the complete loop set.
- Training and scoring rely on 23 chromosome-wise models to avoid predicting loops from the same map used for training; transfer still assumes the loop interaction pattern generalizes across cell types and species.

## Evidence

- [results] Cross-cell-type transferability of CTCF model: [evidence span withheld — anchor status: not_found]
- [readme] Depth-based model selection: "depth               Calculate the total number of intra-chromosomal chromatin contacts and select the
                            most appropriate pre-trained model for you."
- [abstract] Cross-platform/species generalization: "We show that our models perform well in different platforms, across different sequencing depths, and across different species."
- [methods] Pool function role: "The pool function serves to select the most significant non-redundant results from per-pixel probabilities calculated by the score functions"
- [readme] Normalization weight requirement: "The name of the weight column in your Cooler URI for normalizing the contact
                            signals. Specify it to "raw" if you want to use the raw signals."
