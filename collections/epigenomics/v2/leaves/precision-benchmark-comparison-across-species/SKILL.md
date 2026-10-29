---
name: precision-benchmark-comparison-across-species
description: Use when mapping regulatory-element intervals (ChIP-seq, DNase-seq, methylation)
  across species and you need to quantify mapping precision against matched orthologous
  epigenome datasets, comparing AdaLiftOver with UCSC liftOver baselines.
when_to_use_negative:
- When no orthologous/matched epigenome datasets are available to train the logistic
  regression parameters or to serve as ground truth for precision computation
- When the query regions are protein-coding or coding-sequence intervals rather than
  non-coding regulatory regions, which is outside AdaLiftOver's stated scope
- When only a chain-file-based mapping suffices and no epigenomic repertoire is available
  — plain UCSC liftOver is the appropriate baseline, not this benchmarking skill
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: AdaLiftOver
  role: Performs the adaptive liftOver mapping with epigenomic signal and sequence
    grammar similarity scoring under LOOCV; installed via devtools::install_github("ThomasDCY/AdaLiftOver")
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Baseline method whose precision is compared against AdaLiftOver, run with
    reciprocal-best chain files
- name: rtracklayer
  role: Imports UCSC reciprocal-best chain files (e.g. mm10.hg38.rbest.chain, rn6.hg38.rbest.chain)
    for both AdaLiftOver and liftOver baselines
- name: GenomicRanges
  role: Operates on query and candidate target genomic intervals as GRanges/GRangesList
    objects
- name: data.table
  role: Fast data manipulation; rbindlist aggregates per-tissue training_module results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:dfbace8f016d065a8e45d72fc40b6de8826a5e73f124b30fe3a2f78fa8457a78
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# precision-benchmark-comparison-across-species

## Summary

Benchmark AdaLiftOver against UCSC liftOver using leave-one-out cross-validation (LOOCV) over matched human–species epigenome datasets, computing per-species mapping precision to quantify the precision gain of adaptive epigenome- and grammar-aware liftOver.

## When to use

Use when mapping regulatory-element intervals (ChIP-seq, DNase-seq, methylation) across species and you need to quantify mapping precision against matched orthologous epigenome datasets, comparing AdaLiftOver with UCSC liftOver baselines.

## When NOT to use

- When no orthologous/matched epigenome datasets are available to train the logistic regression parameters or to serve as ground truth for precision computation
- When the query regions are protein-coding or coding-sequence intervals rather than non-coding regulatory regions, which is outside AdaLiftOver's stated scope
- When only a chain-file-based mapping suffices and no epigenomic repertoire is available — plain UCSC liftOver is the appropriate baseline, not this benchmarking skill

## Inputs

- Matched human–species functional genomic datasets (TF/DNA methylation ChIP-seq and DNase-seq peaks) for chicken, cow, horse, pig, rat, and zebrafish, organized as GRangesList objects (e.g. epigenome_rn6_test, epigenome_hg38_test)
- UCSC reciprocal-best chain files (e.g. rn6.hg38.rbest.chain.gz) imported via rtracklayer::import.chain
- Species-specific BSgenome objects (e.g. BSgenome.Rnorvegicus.UCSC.rn6) for sequence grammar similarity
- JASPAR PWM list (jaspar_pfm_list) for motif scanning

## Outputs

- Scored and filtered candidate target region lists per query region (gr_candidate / gr_list_filter from gr_candidate_filter)
- Per-species mapping precision table for AdaLiftOver and UCSC liftOver
- Relative precision-boost comparison (expected 17–951% gain, average precision 0.60–0.82 across six species)

## How to apply

Load the matched human–species functional genomic datasets (TF/DNA methylation ChIP-seq and DNase-seq) for each model organism together with the UCSC reciprocal-best chain files (e.g. rn6.hg38.rbest.chain.gz). Run AdaLiftOver LOOCV: map each species epigenome dataset to human with adaptive_liftover while excluding the current dataset from epigenomic signal similarity calculations, then compute epigenomic signal and sequence grammar similarities, aggregate them as logistic probability scores, and filter candidates with gr_candidate_filter (best_k and score threshold; parameters learned with training_module). As the baseline, map the same query regions with UCSC liftOver using the reciprocal-best chain files. Compute mapping precision per species for both methods against the matched orthologous human datasets, then compare per-species to verify the expected average AdaLiftOver precision of 0.60–0.82 and a 17–951% relative precision boost over UCSC liftOver.

## Related tools

- **AdaLiftOver** (Performs the adaptive liftOver mapping with epigenomic signal and sequence grammar similarity scoring under LOOCV; installed via devtools::install_github("ThomasDCY/AdaLiftOver")) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Baseline method whose precision is compared against AdaLiftOver, run with reciprocal-best chain files)
- **rtracklayer** (Imports UCSC reciprocal-best chain files (e.g. mm10.hg38.rbest.chain, rn6.hg38.rbest.chain) for both AdaLiftOver and liftOver baselines)
- **GenomicRanges** (Operates on query and candidate target genomic intervals as GRanges/GRangesList objects)
- **data.table** (Fast data manipulation; rbindlist aggregates per-tissue training_module results)

## Examples

```
chain <- rtracklayer::import.chain('rn6.hg38.rbest.chain'); gr_list <- adaptive_liftover(gr, chain); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- Per-species average AdaLiftOver precision must fall within 0.60–0.82 across the six model organisms (chicken, cow, horse, pig, rat, zebrafish)
- Relative precision gain over UCSC liftOver must be positive and within the reported 17–951% range per species
- In LOOCV, the currently mapped dataset must be excluded from epigenomic signal similarity calculations (no self-overlap leakage)
- gr_candidate_filter must retain at most best_k candidate target regions per query region and drop candidates below the score threshold learned by training_module (e.g. max_filter_proportion = 0.5)
- Candidate target regions labeled positive must overlap the corresponding human orthologous epigenome peaks; negatives must not

## Limitations

- For model organisms other than mouse, the ENCODE human–mouse repertoire does not apply; an orthologous epigenome repertoire must be collected from scratch and logistic regression parameters re-estimated with training_module
- Existing sequence-conservation methods (including the UCSC liftOver baseline) suffer from multi-mapping, low precision, and low mapping rate issues
- The repository has no changelog and no discussion section content available for tracking version-specific behavior

## Evidence

- [intro] LOOCV exclusion of the current dataset: "To conduct LOOCV, we first apply AdaLiftOver to map each rat epigenome dataset to human while excluding the current dataset for epigenomic signal similarity calculations."
- [results] Precision boost over UCSC liftOver: "with an average precision of 0.60–0.82 across the species, AdaLiftOver boosts the precision by an average of 17–951% compared to UCSC liftOver (Fig. 4d)."
- [intro] Training module for non-mouse organisms: "AdaLiftOver provides a training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
- [readme] Chain file import via rtracklayer: "chain <- rtracklayer::import.chain('rn6.hg38.rbest.chain')"
