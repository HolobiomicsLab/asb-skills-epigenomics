---
name: candidate-target-region-filtering-with-threshold
description: Use when a GRangesList of candidates already carries epigenomic and sequence-grammar
  similarity scores per query region and must be reduced to high-confidence orthologous
  target regions before downstream use.
when_to_use_negative:
- The candidate list has not yet been scored with epigenomic signal and sequence grammar
  similarities — filtering unscored candidates would discard information arbitrarily
- No training pair of matched epigenome datasets is available and defaults from another
  organism are not validated — the threshold may filter out too many candidate target
  regions
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: AdaLiftOver
  role: Provides the gr_candidate_filter() function and the training_module() for
    estimating logistic regression parameters and suggesting the score threshold
  repo: https://github.com/keleslab/AdaLiftOver
- name: R
  role: Language in which AdaLiftOver is implemented and gr_candidate_filter() is
    invoked
- name: GenomicRanges
  role: Data structure (GRanges/GRangesList) holding the candidate target regions
    that are filtered
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:daed999811aec12bc7b472d1d7af4043052469115509b50089d43556823b0cfa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/keleslab/AdaLiftOver
  tools:
  - AdaLiftOver
  - GenomicRanges
  - Matrix
  - R
  - data.table
  - motifmatchr
  - rtracklayer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# candidate-target-region-filtering-with-threshold

## Summary

Final step of the AdaLiftOver pipeline: apply the gr_candidate_filter() function to the scored candidate target region list so that each query region keeps at most best_k orthologous candidates whose logistic probability scores exceed a threshold.

## When to use

Use when a GRangesList of candidates already carries epigenomic and sequence-grammar similarity scores per query region and must be reduced to high-confidence orthologous target regions before downstream use.

## When NOT to use

- The candidate list has not yet been scored with epigenomic signal and sequence grammar similarities — filtering unscored candidates would discard information arbitrarily
- No training pair of matched epigenome datasets is available and defaults from another organism are not validated — the threshold may filter out too many candidate target regions

## Inputs

- Scored GRangesList of candidate target regions per query region (output of compute_similarity_epigenome and compute_similarity_grammar)
- best_k parameter (integer, max candidates retained per query region)
- threshold parameter (logistic probability score cutoff)
- Optionally: labels from a pair of matched epigenome datasets (e.g. gr_candidate_list and epigenome_hg38_test) for training_module to estimate parameters and threshold

## Outputs

- Filtered GRangesList (e.g. gr_list_filter) with at most best_k scored candidate target regions per query region, all above the threshold

## How to apply

After adaptive_liftover(), compute_similarity_epigenome() (using matched orthologous ENCODE functional genomic datasets) and compute_similarity_grammar() (motif scanning with motifmatchr against jaspar_pfm_list) have aggregated logistic probability scores for each candidate, pass the scored list to gr_candidate_filter(). Set best_k to the maximum number of candidate target regions retained per query region (the README vignette uses best_k = 1L) and set threshold on the logistic probability score (e.g. 0.5) so candidates below it are dropped. Choose the threshold via the training module — AdaLiftOver's training_module() estimates logistic regression parameters from a pair of matched epigenome datasets (e.g. epigenome_rn6_test vs epigenome_hg38_test, with max_filter_proportion = 0.5 and interaction = FALSE) and suggests an optimal score threshold without filtering out too many candidate target regions. For a new organism such as rat, run training across tissues, then average the logistic regression parameters as defaults.

## Related tools

- **AdaLiftOver** (Provides the gr_candidate_filter() function and the training_module() for estimating logistic regression parameters and suggesting the score threshold) — https://github.com/keleslab/AdaLiftOver
- **R** (Language in which AdaLiftOver is implemented and gr_candidate_filter() is invoked)
- **GenomicRanges** (Data structure (GRanges/GRangesList) holding the candidate target regions that are filtered)

## Examples

```
gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- Output is a GRangesList in which each query region has at most best_k candidate target regions
- All retained candidates have logistic probability scores >= the chosen threshold (e.g. 0.5 in the README example)
- The filter does not remove too many candidate target regions — validated by training_module's suggested optimal threshold with max_filter_proportion (e.g. 0.5)
- Candidates labelled positive (overlapping corresponding human epigenome peaks in LOOCV) are enriched among retained regions versus negatives
- Averaged logistic regression parameters across tissues serve as sensible defaults for the organism (e.g. rat studies)

## Limitations

- Threshold selection depends on having a training pair of matched orthologous epigenome datasets; for organisms other than mouse, parameters must be learned or averaged from a LOOCV example
- max_filter_proportion and the interaction setting in training_module affect the suggested threshold; filtering too aggressively can remove valid candidates
- Filtering assumes scores reflect conservation of epigenomic signals and sequence grammars; datasets excluded during LOOCV change similarity calculations

## Evidence

- [intro] We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function.: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
- [intro] best_k and threshold parameters: "**best_k**: retain at most best k candidate target regions for each query region."
- [readme] Training module suggests threshold: "AdaLiftOver provides a handy training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
- [readme] Scored and filtered output: "For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information."
- [readme] LOOCV labeling: "we first label the candidate target regions in the human genome as positives if they overlap with the corresponding human epigenome peaks and as negatives otherwise."
