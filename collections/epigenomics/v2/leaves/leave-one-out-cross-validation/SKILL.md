---
name: leave-one-out-cross-validation
description: Use when benchmarking cross-species regulatory-element mapping accuracy
  with matched orthologous functional genomic datasets (e.g. 67 matched human-mouse
  ENCODE ChIP-seq/DNase-seq sets), where each dataset must be held out in turn.
when_to_use_negative:
- When only one query dataset is available per species pair, so excluding it would
  leave no epigenomic signal repertoire for similarity calculations
- When the goal is de novo mapping without matched orthologous epigenome data to serve
  as LOOCV ground truth
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: AdaLiftOver
  role: 'Adaptive liftOver method under test: maps query regions via UCSC chain and
    prioritizes candidates with epigenomic/grammar similarity scores'
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Baseline mapper used in the LOOCV precision comparison across species
- name: GenomicRanges
  role: Operates genomic intervals for query regions, candidate targets, and orthologous-overlap
    classification
- name: data.table
  role: Fast data manipulation and aggregation of LOOCV fold results (rbindlist over
    tissues)
- name: rtracklayer
  role: Imports UCSC chain files (import.chain) used for liftOver-based candidate
    generation
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:2e7f54a675156e347dabcc1a4ae87b7538ac48dc7020077c171d392a15ddf4c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/keleslab/AdaLiftOver
  tools:
  - AdaLiftOver
  - GenomicRanges
  - R
  - UCSC liftOver
  - data.table
  - rtracklayer
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# leave-one-out-cross-validation

## Summary

A LOOCV protocol for benchmarking AdaLiftOver: for each matched epigenome dataset, map its query regions to the orthologous genome while excluding that dataset from epigenomic signal similarity calculations, then score precision/AUROC/AUPR across all folds.

## When to use

Use when benchmarking cross-species regulatory-element mapping accuracy with matched orthologous functional genomic datasets (e.g. 67 matched human-mouse ENCODE ChIP-seq/DNase-seq sets), where each dataset must be held out in turn.

## When NOT to use

- When only one query dataset is available per species pair, so excluding it would leave no epigenomic signal repertoire for similarity calculations
- When the goal is de novo mapping without matched orthologous epigenome data to serve as LOOCV ground truth

## Inputs

- Matched orthologous functional genomic datasets (TF/DNA methylation ChIP-seq and DNase-seq peaks), e.g. 67 human-mouse ENCODE epigenomes as GRangesList epigenome_mm10/epigenome_hg38
- UCSC reciprocal-best chain file (e.g. mm10.hg38.rbest.chain, rn6.hg38.rbest.chain.gz) imported via rtracklayer::import.chain
- Query regions (GRanges peaks) from each held-out dataset
- JASPAR PWM list (jaspar_pfm_list) for sequence grammar similarity
- BSgenome objects for both species (e.g. BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38)

## Outputs

- Per-fold classification of top-scored candidate target regions as correct/incorrect by overlap with matched orthologous regions
- Per-window-size AUROC and AUPR summary table (loocv_results.csv) with mean and standard deviation across folds
- Per-species mapping precision for AdaLiftOver vs UCSC liftOver

## How to apply

For each fold, run AdaLiftOver's adaptive_liftover with the reciprocal-best chain file (e.g. mm10.hg38.rbest.chain) while excluding the held-out dataset from the epigenome repertoire used for signal similarity, so the held-out orthologous peaks serve as an unbiased ground truth. Compute epigenomic signal and sequence grammar (JASPAR motif) similarities, aggregate them as logistic probability scores, and filter candidates with gr_candidate_filter (best_k and score threshold). Classify the top-scored candidate as correct if it overlaps the matched orthologous region of the held-out dataset. For window-size tuning, repeat across 0–5 kb in 400 bp increments (13 sizes) and compute AUROC/AUPR per window across all 67 folds; for cross-species benchmarking, compute per-species precision against the UCSC liftOver baseline.

## Related tools

- **AdaLiftOver** (Adaptive liftOver method under test: maps query regions via UCSC chain and prioritizes candidates with epigenomic/grammar similarity scores) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Baseline mapper used in the LOOCV precision comparison across species)
- **GenomicRanges** (Operates genomic intervals for query regions, candidate targets, and orthologous-overlap classification)
- **data.table** (Fast data manipulation and aggregation of LOOCV fold results (rbindlist over tissues))
- **rtracklayer** (Imports UCSC chain files (import.chain) used for liftOver-based candidate generation)

## Examples

```
chain <- rtracklayer::import.chain('rn6.hg38.rbest.chain'); gr_list <- adaptive_liftover(gr, chain); training_result <- training_module(gr_candidate_list[[tissue]], epigenome_hg38_test[[tissue]], max_filter_proportion = 0.5, interaction = FALSE)
```

## Evaluation signals

- Window-size grid: mean AUROC ≈ 0.820 ± 0.0113 and AUPR ≈ 0.604 ± 0.0371 attained at the ~2 kb local window across the 67 human-mouse folds
- Per-species average precision falls within 0.60–0.82 across the six model organisms (chicken, cow, horse, pig, rat, zebrafish)
- AdaLiftOver precision boost over UCSC liftOver is positive and within 17–951% per species
- Each fold's top-scored candidate is scored correct only if it overlaps the matched orthologous region of the held-out dataset
- Held-out dataset is verifiably excluded from epigenome repertoire used for signal similarity in each fold

## Limitations

- Requires a matched orthologous epigenome repertoire; for organisms beyond mouse, logistic regression parameters and score thresholds must be re-estimated via the training_module
- Precision depends on chain-file quality (reciprocal-best chains) and on gr_candidate_filter settings (best_k, threshold)
- LOOCV excludes one dataset at a time, so results assume the remaining repertoire is representative of the species' epigenomic diversity

## Evidence

- [intro] To conduct LOOCV, we first apply AdaLiftOver to map each rat epigenome dataset to human while excluding the current dataset for epigenomic signal similarity calculations.: "To conduct LOOCV, we first apply AdaLiftOver to map each rat epigenome dataset to human while excluding the current dataset for epigenomic signal similarity calculations."
- [results] Optimizing the area under the ROC and PR curves across the LOOCV grid yielded ~2 kb as the optimal window size (AUROC: 0.820 ± 0.0113, AUPR: 0.604 ± 0.0371): [evidence span withheld — anchor status: not_found]
- [results] with an average precision of 0.60–0.82 across the species, AdaLiftOver boosts the precision by an average of 17–951% compared to UCSC liftOver: "with an average precision of 0.60–0.82 across the species, AdaLiftOver boosts the precision by an average of 17–951% compared to UCSC liftOver"
- [intro] We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function.: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
- [readme] we first label the candidate target regions in the human genome as positives if they overlap with the corresponding human epigenome peaks and as negatives otherwise.: "we first label the candidate target regions in the human genome as positives if they overlap with the corresponding human epigenome peaks and as negatives otherwise."
