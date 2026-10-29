---
name: cross-species-epigenomic-region-mapping
description: Use when mapping regulatory peaks (ChIP-seq, DNase-seq, ATAC-seq) across
  species and sequence-based liftOver alone yields multi-mappers or low precision,
  and matched orthologous epigenomic datasets are available for both species.
when_to_use_negative:
- No matched orthologous epigenomic datasets (TF/DNA methylation ChIP-seq or DNase-seq)
  exist in both species — epigenomic signal similarity cannot be computed and default
  parameters may not transfer.
- No UCSC chain file or BSgenome object is available for the species pair and genome
  builds in question.
- The goal is coding-sequence or whole-genome syntenic alignment rather than prioritizing
  orthologous non-coding regulatory elements.
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3452
tools:
- name: AdaLiftOver
  role: 'Core R package: adaptive_liftover() extends queries via the UCSC liftOver
    framework; compute_similarity_epigenome() and compute_similarity_grammar() score
    candidates; gr_candidate_filter() applies best_k/threshold; training_module()
    refits logistic parameters.'
  repo: https://github.com/keleslab/AdaLiftOver
- name: UCSC liftOver
  role: Chain-based framework underlying adaptive_liftover(); reciprocal-best chain
    files (e.g., mm10.hg38.rbest.chain.gz) define the candidate target regions and
    serve as the precision baseline.
- name: rtracklayer
  role: Imports the UCSC reciprocal-best chain file via import.chain() before mapping.
- name: motifmatchr
  role: Fast motif scanning used by compute_similarity_grammar() to compute sequence
    grammar similarity between query and candidate regions.
  repo: https://github.com/GreenleafLab/motifmatchr
- name: GenomicRanges
  role: Holds query peaks and matched epigenome peak sets as GRanges/GRangesList for
    interval operations and positive/negative overlap labeling.
- name: data.table
  role: Fast data manipulation; rbindlist() aggregates training_module() results across
    tissues for species-level default parameters.
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad149
    title: 'AdaLiftOver: high-resolution identification of orthologous regulatory
      elements with Adaptive liftOver'
  source_content_hash: sha256:ea975a64f9c0c08b6b4a7b06181b7273ece8d59cbe4d188a6ee670670234e9d9
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
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cross-species-epigenomic-region-mapping

## Summary

Cross-species identification of orthologous non-coding regulatory elements: query peaks are extended via the UCSC liftOver framework and candidate target regions are prioritized by aggregating epigenomic signal and sequence grammar similarities into logistic probability scores. It counters the multi-mapping, low precision, and low mapping rate of sequence-only conservation methods.

## When to use

Use when mapping regulatory peaks (ChIP-seq, DNase-seq, ATAC-seq) across species and sequence-based liftOver alone yields multi-mappers or low precision, and matched orthologous epigenomic datasets are available for both species.

## When NOT to use

- No matched orthologous epigenomic datasets (TF/DNA methylation ChIP-seq or DNase-seq) exist in both species — epigenomic signal similarity cannot be computed and default parameters may not transfer.
- No UCSC chain file or BSgenome object is available for the species pair and genome builds in question.
- The goal is coding-sequence or whole-genome syntenic alignment rather than prioritizing orthologous non-coding regulatory elements.

## Inputs

- Query regions as GRanges (e.g., ChIP-seq, DNase-seq, or ATAC-seq peaks) in the source species genome build (e.g., mm10, rn6)
- UCSC reciprocal-best chain file for the species pair (e.g., mm10.hg38.rbest.chain.gz, rn6.hg38.rbest.chain.gz)
- Matched orthologous epigenomic datasets organized as GRangesList (e.g., epigenome_mm10/epigenome_hg38, epigenome_rn6_test/epigenome_hg38_test)
- BSgenome objects for both species (e.g., BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38, BSgenome.Rnorvegicus.UCSC.rn6)
- jaspar_pfm_list motif PWM list for sequence grammar similarity

## Outputs

- Scored and filtered list of candidate target regions per query region (GRanges/GRangesList with logistic probability scores)
- Logistic regression parameters and suggested optimal score threshold from training_module() (e.g., training_result data.table)
- AUROC/AUPR evaluation metrics and scored candidate tables when labeled matched data exist (e.g., islet_liftover_evaluation.csv, adaliftover_group_scores.csv)

## How to apply

Download a UCSC reciprocal-best chain file for the species pair (e.g., mm10.hg38.rbest.chain.gz or rn6.hg38.rbest.chain.gz), import it with rtracklayer::import.chain(), and run adaptive_liftover(gr, chain) to extend query regions via the liftOver framework and enumerate candidate target regions; BSgenome objects for both builds (e.g., BSgenome.Mmusculus.UCSC.mm10, BSgenome.Hsapiens.UCSC.hg38) must be installed. Compute epigenomic signal similarities with compute_similarity_epigenome() over matched orthologous datasets held as GRangesList (e.g., the 67 matched ENCODE human-mouse TF/DNA methylation ChIP-seq and DNase-seq datasets, or epigenome_rn6_test/epigenome_hg38_test for rat), and compute sequence grammar similarities with compute_similarity_grammar() using jaspar_pfm_list and motifmatchr. Aggregate both similarity types as logistic probability scores and apply gr_candidate_filter() (e.g., best_k = 1L, threshold = 0.5) to retain at most best_k candidates per query above the score threshold. For organisms other than mouse, refit the logistic regression parameters with training_module() on labeled pairs (positives = candidates overlapping the matched orthologous peaks, negatives otherwise; max_filter_proportion = 0.5 in the rat example) and optionally average parameters across tissues as defaults; the mouse default (trained via LOOCV on the 67 ENCODE datasets) transferred to islet data with nearly identical performance (AUROC 0.880 vs 0.879 refit). In LOOCV benchmarks, exclude the dataset being mapped from epigenomic signal similarity calculations to avoid self-support.

## Related tools

- **AdaLiftOver** (Core R package: adaptive_liftover() extends queries via the UCSC liftOver framework; compute_similarity_epigenome() and compute_similarity_grammar() score candidates; gr_candidate_filter() applies best_k/threshold; training_module() refits logistic parameters.) — https://github.com/keleslab/AdaLiftOver
- **UCSC liftOver** (Chain-based framework underlying adaptive_liftover(); reciprocal-best chain files (e.g., mm10.hg38.rbest.chain.gz) define the candidate target regions and serve as the precision baseline.)
- **rtracklayer** (Imports the UCSC reciprocal-best chain file via import.chain() before mapping.)
- **motifmatchr** (Fast motif scanning used by compute_similarity_grammar() to compute sequence grammar similarity between query and candidate regions.) — https://github.com/GreenleafLab/motifmatchr
- **GenomicRanges** (Holds query peaks and matched epigenome peak sets as GRanges/GRangesList for interval operations and positive/negative overlap labeling.)
- **data.table** (Fast data manipulation; rbindlist() aggregates training_module() results across tissues for species-level default parameters.)

## Examples

```
chain <- rtracklayer::import.chain("mm10.hg38.rbest.chain"); gr_list <- adaptive_liftover(gr, chain); gr_list <- compute_similarity_epigenome(gr, gr_list, epigenome_mm10, epigenome_hg38)
```

## Evaluation signals

- gr_candidate_filter() output must retain at most best_k candidate target regions per query region, all with scores at or above the threshold (e.g., best_k = 1L, threshold = 0.5).
- Mapping mouse pancreatic islet ATAC-seq peaks with default LOOCV-trained parameters should reproduce AUROC ≈ 0.880 / AUPR ≈ 0.879, closely matching the islet-refit parameters (0.879 / 0.878).
- Six-species LOOCV (chicken, cow, horse, pig, rat, zebrafish) should yield average AdaLiftOver precision of 0.60–0.82 and relative precision gains of 17–951% over UCSC liftOver with reciprocal-best chains.
- In LOOCV, the dataset being mapped must be excluded from epigenomic signal similarity calculations; leaving it in inflates scores.
- Every scored candidate must aggregate both an epigenomic signal similarity and a sequence grammar similarity into a single logistic probability score.

## Limitations

- For model organisms other than mouse, the orthologous epigenome repertoire must be collected from scratch and parameters refit with training_module(); defaults for such species are averaged across tissues (e.g., the rat training_result).
- Prioritization can override sequence-based mappability: 16.4% of Group I zebrafish brain ATAC-seq regions uniquely mapped by sequence conservation received AdaLiftOver scores < 0.5, reducing raw mapping rate in favor of regulatory support.
- Default parameters are validated on mouse-human ENCODE data (islet AUROC 0.880 vs 0.879 refit); other species pairs may require refitting and a tuned score threshold that does not filter out too many candidates (max_filter_proportion = 0.5).
- Baseline chain-based liftOver suffers multi-mapping, low precision, and low mapping rate issues; gr_candidate_filter settings trade recall for precision.

## Evidence

- [abstract] AdaLiftOver identifies functionally similar orthologous non-coding regions across species.: "We develop Adaptive liftOver (AdaLiftOver), a large-scale computational tool for identifying functionally similar orthologous non-coding regions across species."
- [abstract] Sequence-only cross-species mapping suffers multi-mapping, low precision, and low mapping rate — the gap this skill fills.: "existing methods for mapping non-coding genomic regions across species are challenged by the multi-mapping, low precision, and low mapping rate issues"
- [intro] Similarities are aggregated as logistic probability scores and filtered with gr_candidate_filter().: "We then aggregate the epigenome signal and sequence grammar similarities as logistic probability scores and filter the scores with gr_candidate_filter() function."
- [intro] LOOCV maps each dataset while excluding it from epigenomic signal similarity calculations.: "To conduct LOOCV, we first apply AdaLiftOver to map each rat epigenome dataset to human while excluding the current dataset for epigenomic signal similarity calculations."
- [readme] training_module() estimates logistic regression parameters and an optimal score threshold.: "AdaLiftOver provides a handy training module to estimate the logistic regression parameters and suggest an optimal score threshold without filtering out too many candidate target regions."
- [readme] Non-mouse organisms require collecting the orthologous epigenome repertoire from scratch.: "For other model organisms, e.g. rats, we will need to collect orthologous epigenome repertoire from scratch."
