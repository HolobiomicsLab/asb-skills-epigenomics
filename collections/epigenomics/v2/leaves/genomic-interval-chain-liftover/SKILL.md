---
name: genomic-interval-chain-liftover
description: Use when lifting non-coding regulatory regions (peaks, GWAS SNP loci)
  between species with a UCSC chain file and candidates must be ranked by functional
  (epigenomic/motif) similarity rather than position alone.
when_to_use_negative:
- Input regions are coding sequences or need only synteny-based positional mapping
  with no functional prioritization — plain UCSC liftOver suffices
- No matched orthologous functional genomic datasets are available for the species
  pair, so epigenomic signal similarities cannot be computed
- Input is already a scored/filtered candidate list — use downstream evaluation or
  training_module instead of re-mapping
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: AdaLiftOver
  role: R package providing adaptive_liftover(), compute_similarity_epigenome(), compute_similarity_grammar(),
    gr_candidate_filter(), and training_module()
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Underlying framework used in the two-pass direct-then-extended mapping of
    query regions
- name: rtracklayer
  role: Imports the UCSC chain file and executes liftOver passes
- name: motifmatchr
  role: Fast motif scanning to compute sequence grammar similarities against jaspar_pfm_list
- name: GenomicRanges
  role: Represents and operates on query and candidate genomic intervals
- name: data.table
  role: Fast data manipulation for similarity computations and training result aggregation
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:f8c5b4a7e7395cacdf4941f4a5d1c7f98b026f5cbb007bc600aac314d426a2ae
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# genomic-interval-chain-liftover

## Summary

Maps genomic intervals (e.g. mouse ATAC-seq peaks) across species using UCSC reciprocal-best chain files, extending failed queries with a local window and prioritizing candidate orthologous target regions by epigenomic signal and sequence grammar similarity. This is the core two-step adaptive mapping mechanism of AdaLiftOver.

## When to use

Use when lifting non-coding regulatory regions (peaks, GWAS SNP loci) between species with a UCSC chain file and candidates must be ranked by functional (epigenomic/motif) similarity rather than position alone.

## When NOT to use

- Input regions are coding sequences or need only synteny-based positional mapping with no functional prioritization — plain UCSC liftOver suffices
- No matched orthologous functional genomic datasets are available for the species pair, so epigenomic signal similarities cannot be computed
- Input is already a scored/filtered candidate list — use downstream evaluation or training_module instead of re-mapping

## Inputs

- Query regions as a GenomicRanges object (e.g. mouse pancreatic islet ATAC-seq peaks, 46,676 regions)
- UCSC reciprocal-best chain file (e.g. mm10.hg38.rbest.chain.gz, rn6.hg38.rbest.chain.gz), unzipped
- Matched orthologous epigenome repertoires as GRangesList objects (e.g. epigenome_mm10, epigenome_hg38, epigenome_rn6_test)
- JASPAR position frequency matrix list (jaspar_pfm_list) and BSgenome objects for both species

## Outputs

- A scored and filtered GRangesList of candidate target regions per query region, with logistic probability scores
- Filtered candidate set retaining at most best_k candidates per query above the score threshold
- Training result table of averaged logistic regression parameters (data.table from training_module())

## How to apply

Load query regions as a GenomicRanges object and import the reciprocal-best chain file (e.g. mm10.hg38.rbest.chain) with rtracklayer::import.chain(). Run adaptive_liftover(gr, chain): a first direct UCSC liftOver pass, then, for regions that fail, re-run liftOver on the query extended with a local window on both sides to generate candidate target regions. Score each query-candidate pair with compute_similarity_epigenome() using matched orthologous ENCODE TF/DNA methylation ChIP-seq and DNase-seq datasets (67 human-mouse sets) and compute_similarity_grammar() via fast motif scanning with motifmatchr against jaspar_pfm_list (BSgenome objects required, e.g. BSgenome.Mmusculus.UCSC.mm10 and BSgenome.Hsapiens.UCSC.hg38). Aggregate similarities into logistic probability scores and apply gr_candidate_filter() with best_k (e.g. best_k = 1L) and a score threshold (e.g. threshold = 0.5) so that at most best_k candidates are retained per query without filtering out too many. For organisms other than mouse, estimate parameters with training_module() on a pair of matched epigenome datasets (max_filter_proportion = 0.5) and average them as defaults; the default mouse-human parameters from LOOCV reproduce islet AUROC 0.880 / AUPR 0.879 versus refit 0.879/0.878.

## Related tools

- **AdaLiftOver** (R package providing adaptive_liftover(), compute_similarity_epigenome(), compute_similarity_grammar(), gr_candidate_filter(), and training_module()) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Underlying framework used in the two-pass direct-then-extended mapping of query regions)
- **rtracklayer** (Imports the UCSC chain file and executes liftOver passes)
- **motifmatchr** (Fast motif scanning to compute sequence grammar similarities against jaspar_pfm_list)
- **GenomicRanges** (Represents and operates on query and candidate genomic intervals)
- **data.table** (Fast data manipulation for similarity computations and training result aggregation)

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list_filter <- gr_candidate_filter(gr_list, best_k = 1L, threshold = 0.5)
```

## Evaluation signals

- Output is a scored, filtered list of candidate target regions per query region whose scores pass the suggested threshold without filtering out too many candidates (best_k and threshold respected)
- Islet case study: default LOOCV parameters give AUROC 0.880 / AUPR 0.879, agreeing with islet-refit parameters (AUROC 0.879 / AUPR 0.878)
- Multi-species LOOCV benchmark: average AdaLiftOver precision within 0.60–0.82 across chicken, cow, horse, pig, rat, zebrafish, with a 17–951% precision boost over UCSC liftOver
- Query regions that fail direct liftOver still receive candidates after local window extension (no silent drops at the first pass)
- Training module returns a suggested optimal score threshold with max_filter_proportion respected (e.g. 0.5)

## Limitations

- Existing sequence-conservation methods (including plain UCSC liftOver) suffer from multi-mapping, low precision, and low mapping rate issues; AdaLiftOver mitigates but depends on availability of matched orthologous epigenome datasets
- Default logistic parameters are trained on the 67 matched human-mouse ENCODE datasets; for other organisms (e.g. rat) parameters must be re-estimated with training_module() and averaged
- Chain files must be reciprocal-best and unzipped for rtracklayer import; BSgenome packages for both genomes are required for grammar similarity

## Evidence

- [intro] For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information.: "For each query region, AdaLiftOver outputs a scored and filtered list of candidate target regions that are most similar to the query region in terms of regulatory information."
- [other] AdaLiftOver first applies UCSC liftOver directly to the query region; if that fails, it extends the query region with a local window on both sides and applies UCSC liftOver again.: [evidence span withheld — anchor status: not_found]
- [intro] **best_k**: retain at most best k candidate target regions for each query region.: "**best_k**: retain at most best k candidate target regions for each query region."
- [readme] chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"): "chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain")"
- [readme] AdaLiftOver provides a handy training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions.: "AdaLiftOver provides a handy training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
