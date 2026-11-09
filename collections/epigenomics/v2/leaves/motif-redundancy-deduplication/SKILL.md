---
name: motif-redundancy-deduplication
description: Use when constructing a TF-target gene prior from motif scans of open-chromatin
  peaks and same-family TFs yield overlapping motif-to-peak sets that would create
  redundant, correlated rows in the decoder's TF-gene matrix.
when_to_use_negative:
- The analysis requires resolving which specific TF within a shared-motif family is
  active — deduplication merges same-family motifs and cannot disambiguate them, so
  motif-level activities may encompass multiple TFs.
- The downstream model needs activation-vs-repression directionality or co-binding
  patterns (e.g. AP-1–IRF complexes) — the deduplicated binary prior represents neither.
- Input motif-to-peak assignments are already deduplicated or derived from ChIP-seq
  evidence rather than motif scanning, so pairwise Jaccard collapsing is unnecessary.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: FIMO
  role: Scans ATAC-seq peak sequences against the cis-BP motif reference at P < 1e-5
    to produce the motif-to-peak assignments whose overlaps are measured for deduplication
- name: Python
  role: Implements the pairwise Jaccard similarity computation and motif-pair removal
    filtering
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: Pandas
  role: Handles the motif-peak assignment table and assembly of the final binary matrix
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: ChIPpeakAnno
  role: Applies the nearest-distance rule to assign each deduplicated motif-bearing
    peak to its nearest gene before binarization
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:3ff1df8c5544720e5209609750fbd84cc0fca7b4c8ee838a4541fcb3c07e7ec4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/osmanbeyoglulab/CITRUS
  tools:
  - CITRUS
  - ChIPpeakAnno
  - FIMO
  - Numpy
  - Pandas
  - Python
  - SciPy
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# motif-redundancy-deduplication

## Summary

Deduplicates TF motifs by removing near-identical motif pairs (Jaccard similarity > 0.5) after a FIMO/cis-BP scan of the TCGA pan-cancer ATAC-seq peak atlas, collapsing to a non-redundant set of 320 TF motifs used to constrain CITRUS's TF-to-gene decoder layer.

## When to use

Use when constructing a TF-target gene prior from motif scans of open-chromatin peaks and same-family TFs yield overlapping motif-to-peak sets that would create redundant, correlated rows in the decoder's TF-gene matrix.

## When NOT to use

- The analysis requires resolving which specific TF within a shared-motif family is active — deduplication merges same-family motifs and cannot disambiguate them, so motif-level activities may encompass multiple TFs.
- The downstream model needs activation-vs-repression directionality or co-binding patterns (e.g. AP-1–IRF complexes) — the deduplicated binary prior represents neither.
- Input motif-to-peak assignments are already deduplicated or derived from ChIP-seq evidence rather than motif scanning, so pairwise Jaccard collapsing is unnecessary.

## Inputs

- Pan-cancer ATAC-seq open-chromatin peak atlas (~200K peaks) from the TCGA Genomic Data Commons ATACseq-AWG release
- TF motif reference: MEME-curated cis-BP motif library
- TCGA gene expression matrix across 17 tumor types (used for the >= 50% expressed-samples TF filter)
- FIMO scan output mapping TF motifs to open-chromatin peaks

## Outputs

- Deduplicated set of 320 TF motifs (motif pairs with Jaccard similarity > 0.5 removed)
- Binary {0,1} TF-target gene matrix of shape 320 TF motifs x 5541 filtered genes, packaged as tf_target_matrix.pkl

## How to apply

First scan all pan-cancer ATAC-seq peak sequences (~200K peaks from the GDC ATACseq-AWG release) with FIMO against the MEME-curated cis-BP motif reference at significance threshold P < 1e-5, retaining only TFs whose genes are expressed in >= 50% of samples in at least one of the 17 TCGA tumor types. Then compute pairwise Jaccard similarity between the sets of motif-bearing peaks for every TF motif pair, and remove near-identical motif pairs with Jaccard similarity > 0.5 to yield the final non-redundant set of 320 TF motifs. Rationale: TFs of the same family share similar motifs, so keeping both would inject duplicate latent TF nodes into CITRUS's hidden TF layer and inflate the 320 x 5541 binary TF-target matrix. After deduplication, assign each motif-bearing peak to its nearest gene via the ChIPpeakAnno nearest-distance rule and binarize into the {0,1} 320 x 5541 tf_target_matrix.pkl.

## Related tools

- **FIMO** (Scans ATAC-seq peak sequences against the cis-BP motif reference at P < 1e-5 to produce the motif-to-peak assignments whose overlaps are measured for deduplication)
- **Python** (Implements the pairwise Jaccard similarity computation and motif-pair removal filtering) — https://github.com/osmanbeyoglulab/CITRUS
- **Pandas** (Handles the motif-peak assignment table and assembly of the final binary matrix) — https://github.com/osmanbeyoglulab/CITRUS
- **ChIPpeakAnno** (Applies the nearest-distance rule to assign each deduplicated motif-bearing peak to its nearest gene before binarization)

## Examples

```
fimo --thresh 1e-5 cis-bp.meme tcga_atac_peaks.fa
```

## Evaluation signals

- Final deduplicated motif set contains exactly 320 TF motifs.
- Resulting TF-target matrix has the expected 320 x 5541 dimensions and contains only {0,1} entries.
- All pairwise Jaccard similarity values among retained TF motifs are <= 0.5 (no pair above the removal threshold survived).
- Every retained TF's gene is expressed in >= 50% of samples in at least one of the 17 TCGA tumor types.
- tf_target_matrix.pkl loads cleanly (pickle) and can be supplied as the decoder constraint ('tf_gene') in CITRUS training.

## Limitations

- TFs of the same family often share a similar motif and thus are difficult to disambiguate; TF motifs may encompass the activities of multiple TFs.
- Co-binding TF binding patterns (e.g. AP-1–IRF complexes) are biologically meaningful but not represented in the model.
- No directionality (activation vs repression) is represented in the TF–target gene priors, so deduplication preserves only membership, not regulatory sign.

## Evidence

- [other] Deduplicate near-identical motifs by removing TF motif pairs with Jaccard similarity > 0.5, yielding the final set of 320 TF motifs.: [evidence span withheld — anchor status: not_found]
- [other] FIMO scan threshold: [evidence span withheld — anchor status: not_found]
- [supplementary] GDC ATAC-seq source: "ATAC-seq data are available in the public repository Genomic Data Commons (https://gdc.cancer.gov/about-data/publications/ATACseq-AWG)."
- [discussion] motif rationale from discussion: "TFs of the same family often share a similar motif and thus are difficult to disambiguate. Therefore, TF motifs may encompass the activities of multiple TFs."
