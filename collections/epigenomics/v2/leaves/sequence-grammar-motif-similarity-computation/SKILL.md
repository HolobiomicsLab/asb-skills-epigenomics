---
name: sequence-grammar-motif-similarity-computation
description: 'Use when scoring candidate orthologous target regions after adaptive_liftover:
  you need motif-based sequence grammar similarity between query (e.g. mm10) and target
  (e.g. hg38) intervals to feed AdaLiftOver''s logistic scoring.'
when_to_use_negative:
- Candidate target regions have not been generated yet — adaptive_liftover must run
  first, since there are no target intervals to motif-scan.
- Only epigenomic signal similarity is needed for the comparison and grammar scores
  were already computed on the same candidates; re-scanning motifs is redundant.
- No BSgenome package is available for the query or target species build, so region
  sequences cannot be retrieved for motif scanning.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: motifmatchr
  role: Performs fast motif scanning of query and candidate target region sequences
    to compute sequence grammar similarities
  repo: https://github.com/GreenleafLab/motifmatchr
- name: AdaLiftOver
  role: Provides compute_similarity_grammar(), which wraps motif scanning and stores
    grammar similarity in the candidate target region list
  repo: https://github.com/keleslab/AdaLiftOver
- name: BSgenome
  role: Supplies reference genome sequences for both species (BSgenome.Mmusculus.UCSC.mm10,
    BSgenome.Hsapiens.UCSC.hg38, BSgenome.Rnorvegicus.UCSC.rn6) needed before motif
    scanning
- name: GenomicRanges
  role: Holds the query GRanges and candidate target region list to which grammar
    similarity scores are attached
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:b1589b83fea28d4e17f52928b3c33bf12dd2091dfb4d0be33b261e7cfc1fb9ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreenleafLab/motifmatchr
  tools:
  - AdaLiftOver
  - BSgenome
  - GenomicRanges
  - Matrix
  - R
  - data.table
  - motifmatchr
  - rtracklayer
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# sequence-grammar-motif-similarity-computation

## Summary

Computes sequence grammar similarity between each query region and its candidate target regions via fast motif scanning (motifmatchr) against the Jaspar PFM list shipped with AdaLiftOver. These similarities are aggregated with epigenomic signal similarities into logistic probability scores that prioritize orthologous regulatory elements.

## When to use

Use when scoring candidate orthologous target regions after adaptive_liftover: you need motif-based sequence grammar similarity between query (e.g. mm10) and target (e.g. hg38) intervals to feed AdaLiftOver's logistic scoring.

## When NOT to use

- Candidate target regions have not been generated yet — adaptive_liftover must run first, since there are no target intervals to motif-scan.
- Only epigenomic signal similarity is needed for the comparison and grammar scores were already computed on the same candidates; re-scanning motifs is redundant.
- No BSgenome package is available for the query or target species build, so region sequences cannot be retrieved for motif scanning.

## Inputs

- Query regions GRanges (gr) and candidate target region list (gr_list) returned by adaptive_liftover()
- Jaspar position frequency matrix list (jaspar_pfm_list data object in AdaLiftOver)
- Genome build identifiers for query and target species (e.g., 'mm10', 'hg38', 'rn6')
- BSgenome packages for both species (BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38, BSgenome.Rnorvegicus.UCSC.rn6)

## Outputs

- gr_list updated with a sequence grammar similarity score for each query-candidate target region pair, ready for logistic aggregation with epigenomic similarities

## How to apply

After adaptive_liftover() has produced the gr_list of candidate target regions, load the motif set with data('jaspar_pfm_list') and call compute_similarity_grammar(gr, gr_list, 'mm10', 'hg38', jaspar_pfm_list), passing the query and target genome build strings. The function relies on motifmatchr for fast motif scanning, so the matching BSgenome packages (BSgenome.Mmusculus.UCSC.mm10 and BSgenome.Hsapiens.UCSC.hg38; BSgenome.Rnorvegicus.UCSC.rn6 for rat) must be installed to retrieve region sequences. The returned gr_list carries a sequence grammar similarity for each query-candidate pair, which is then aggregated with epigenomic signal similarities from compute_similarity_epigenome into logistic probability scores before gr_candidate_filter(best_k, threshold). For other species pairs, e.g. rat rn6 vs human hg38 in the LOOCV example, swap the build strings and use the corresponding BSgenome object.

## Related tools

- **motifmatchr** (Performs fast motif scanning of query and candidate target region sequences to compute sequence grammar similarities) — https://github.com/GreenleafLab/motifmatchr
- **AdaLiftOver** (Provides compute_similarity_grammar(), which wraps motif scanning and stores grammar similarity in the candidate target region list) — https://github.com/keleslab/AdaLiftOver
- **BSgenome** (Supplies reference genome sequences for both species (BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38, BSgenome.Rnorvegicus.UCSC.rn6) needed before motif scanning)
- **GenomicRanges** (Holds the query GRanges and candidate target region list to which grammar similarity scores are attached)

## Examples

```
data("jaspar_pfm_list"); gr_list <- compute_similarity_grammar(gr, gr_list, "mm10", "hg38", jaspar_pfm_list)
```

## Evaluation signals

- Returned gr_list preserves the adaptive_liftover structure and now carries a sequence grammar similarity for every query-candidate pair.
- Grammar similarities aggregate with epigenomic signal similarities into logistic probability scores whose candidates pass gr_candidate_filter(best_k, threshold) without filtering out too many candidate target regions.
- In the mouse-islet-to-human case study, default LOOCV parameters with both similarity types reach AUROC 0.880 / AUPR 0.879 (islet-refit 0.879 / 0.878), so large deviations suggest mis-computed grammar similarities.
- Setup check: the README lists the BSgenome packages as required for the examples, so a missing or mismatched genome build (e.g., 'mm10' without BSgenome.Mmusculus.UCSC.mm10) means the step was not applied correctly.

## Limitations

- Sequence grammar similarity alone does not prioritize targets; it must be aggregated with epigenomic signal similarities into logistic probability scores.
- Motif scanning uses the jaspar_pfm_list shipped with AdaLiftOver; the README does not document substituting other motif sets.
- Each new species pair requires its own chain file and BSgenome package (e.g., rn6.hg38.rbest.chain and BSgenome.Rnorvegicus.UCSC.rn6 for rat-to-human).

## Evidence

- [abstract] AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence: "AdaLiftOver builds on the UCSC liftOver framework to extend the query regions and prioritizes the resulting candidate target regions based on the conservation of the epigenomic and the sequence"
- [intro] AdaLiftOver incorporates epigenomic signals as well as sequence grammars to priorize and pinpoint candidate target genomic regions.: "AdaLiftOver incorporates epigenomic signals as well as sequence grammars to priorize and pinpoint candidate target genomic regions."
- [intro] `r CRANpkg("motifmatchr")` is used for fast motif scanning.: "`r CRANpkg("motifmatchr")` is used for fast motif scanning."
- [readme] gr_list <- compute_similarity_grammar(gr, gr_list, "mm10", "hg38", jaspar_pfm_list): "gr_list <- compute_similarity_grammar(gr, gr_list, "mm10", "hg38", jaspar_pfm_list)"
