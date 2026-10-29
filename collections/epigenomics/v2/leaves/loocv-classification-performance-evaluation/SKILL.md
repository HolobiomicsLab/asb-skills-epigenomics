---
name: loocv-classification-performance-evaluation
description: Use when validating AdaLiftOver's candidate target region scoring on
  a repertoire of matched cross-species epigenome datasets (e.
when_to_use_negative:
- When no labeled truth set exists (no held-out human epigenome peaks defining positives/negatives),
  since AUROC/AUPR cannot be computed for the LOOCV comparison
- When only a single epigenome dataset is available — LOOCV requires holding out one
  dataset at a time from the matched repertoire
- For within-species peak-to-peak mapping tasks without a cross-species chain file,
  since adaptive_liftover depends on UCSC liftOver chain-based extension
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: AdaLiftOver
  role: R package providing adaptive_liftover(), compute_similarity_epigenome(), compute_similarity_grammar(),
    gr_candidate_filter(), and the training_module() used to refit logistic regression
    parameters and suggest a score threshold during LOOCV
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Underlying framework used by adaptive_liftover() to extend query regions into
    candidate target regions via the reciprocal-best chain file
- name: rtracklayer
  role: Imports the UCSC chain file with import.chain() before liftOver-based extension
- name: motifmatchr
  role: Performs the fast motif scanning used to compute sequence grammar similarities
    between query and candidate target regions
  repo: https://github.com/GreenleafLab/motifmatchr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:631f40862b2b72806ea26c7d176ebff28315ec3f007246e3829cee5ff7316700
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/keleslab/AdaLiftOver
  tools:
  - AdaLiftOver
  - GenomicRanges
  - Matrix
  - R
  - UCSC liftOver
  - data.table
  - motifmatchr
  - rtracklayer
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# loocv-classification-performance-evaluation

## Summary

Leave-one-out cross-validation protocol for evaluating AdaLiftOver's logistic-probability scoring of candidate orthologous target regions, where each matched epigenome dataset is held out from the epigenomic signal similarity calculations and scored candidates are labeled by overlap with held-out human epigenome peaks, yielding AUROC/AUPR to validate default or refit parameters.

## When to use

Use when validating AdaLiftOver's candidate target region scoring on a repertoire of matched cross-species epigenome datasets (e.g., 67 ENCODE human-mouse TF/DNA methylation ChIP-seq and DNase-seq), or when deciding whether default LOOCV-derived parameters suffice versus refitting the logistic regression on labeled data from a new experiment (e.g., mouse islet ATAC-seq to human).

## When NOT to use

- When no labeled truth set exists (no held-out human epigenome peaks defining positives/negatives), since AUROC/AUPR cannot be computed for the LOOCV comparison
- When only a single epigenome dataset is available — LOOCV requires holding out one dataset at a time from the matched repertoire
- For within-species peak-to-peak mapping tasks without a cross-species chain file, since adaptive_liftover depends on UCSC liftOver chain-based extension

## Inputs

- Query genomic regions (e.g., GRanges of 46,676 mouse pancreatic islet ATAC-seq peaks, mm10)
- Reciprocal-best UCSC chain file (mm10.hg38.rbest.chain.gz)
- Matched human-mouse orthologous epigenome repertoire (67 ENCODE TF/DNA methylation ChIP-seq and DNase-seq datasets as epigenome_mm10/epigenome_hg38 GRangesList)
- Held-out labeled human epigenome peaks per tissue (positives for overlap-based labeling)
- JASPAR PFMs (jaspar_pfm_list) and BSgenome objects (BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38)

## Outputs

- Scored and filtered candidate target regions per query region (gr_list_filter after gr_candidate_filter with best_k and threshold)
- Refitted logistic regression parameters and suggested score threshold from training_module()
- AUROC and AUPR metrics for default vs refit parameter settings (e.g., islet_liftover_evaluation.csv)

## How to apply

Map query regions (e.g., 46,676 mouse pancreatic islet ATAC-seq peaks) with adaptive_liftover() using the mm10.hg38 reciprocal-best UCSC chain at a 2 kb window, then compute epigenomic signal similarities against the 67 matched ENCODE datasets and sequence grammar similarities with motifmatchr over JASPAR PFMs, excluding the dataset currently being evaluated from the epigenomic signal similarity calculations — this exclusion is the defining LOOCV step. Aggregate similarities into logistic probability scores and filter with gr_candidate_filter() using best_k and a score threshold, then label candidate human target regions as positives if they overlap the corresponding held-out human epigenome peaks and negatives otherwise. Score candidates under the default LOOCV-trained parameters, and in parallel refit logistic regression parameters with the AdaLiftOver training_module() on the labeled data (e.g., training_module(gr_candidate, gr_true, max_filter_proportion = 0.5, interaction = FALSE)) and rescore. Compute AUROC and AUPR for both settings; agreement between them (default AUROC 0.880/AUPR 0.879 vs refit 0.879/0.878 on islet data) justifies the default parameter setting.

## Related tools

- **AdaLiftOver** (R package providing adaptive_liftover(), compute_similarity_epigenome(), compute_similarity_grammar(), gr_candidate_filter(), and the training_module() used to refit logistic regression parameters and suggest a score threshold during LOOCV) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Underlying framework used by adaptive_liftover() to extend query regions into candidate target regions via the reciprocal-best chain file)
- **rtracklayer** (Imports the UCSC chain file with import.chain() before liftOver-based extension)
- **motifmatchr** (Performs the fast motif scanning used to compute sequence grammar similarities between query and candidate target regions) — https://github.com/GreenleafLab/motifmatchr

## Examples

```
training_result <- rbindlist(lapply(tissues, function(tissue) training_module(gr_candidate_list[[tissue]], epigenome_hg38_test[[tissue]], max_filter_proportion = 0.5, interaction = FALSE)))
```

## Evaluation signals

- Default LOOCV parameters vs islet-refit parameters agree closely: AUROC 0.880 vs 0.879 and AUPR 0.879 vs 0.878
- The dataset under evaluation is verifiably excluded from epigenomic signal similarity calculations before scoring (leave-one-out invariant)
- Truth labels are defined by overlap: candidate human regions overlapping held-out human epigenome peaks are positives, all others negatives
- training_module() is called with max_filter_proportion = 0.5 so the suggested threshold does not filter out too many candidate target regions
- AUROC and AUPR are both reported so precision behavior at the chosen threshold is assessed alongside ranking quality

## Limitations

- The repository has no changelog and the article's discussion section content is absent, so version-specific behavior of the training module cannot be traced
- Default parameters are derived from the 67 ENCODE human-mouse datasets; for other model organisms (e.g., rat/rn6) the orthologous epigenome repertoire must be collected from scratch and parameters averaged across tissues before use
- Existing sequence-conservation-based cross-species mapping approaches suffer multi-mapping, low precision, and low mapping rate issues, so comparisons limited to such baselines may not reflect performance at high mapping rates

## Evidence

- [intro] LOOCV excluding evaluated dataset from signal similarity: "To conduct LOOCV, we first apply AdaLiftOver to map each rat epigenome dataset to human while excluding the current dataset for epigenomic signal similarity calculations."
- [intro] Training module purpose: "AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
- [readme] Overlap-based labeling of candidates: "we first label the candidate target regions in the human genome as positives if they overlap with the corresponding human epigenome peaks and as negatives otherwise."
- [intro] Filtering candidates with best_k and threshold: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
