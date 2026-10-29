---
name: pretrained-cnn-variant-scoring
description: Use when a set of non-coding SNP coordinates and alleles needs tissue-
  or cell type-specific functional impact scores across ENCODE/Roadmap epigenomic
  tracks, rather than a single pathogenicity score.
when_to_use_negative:
- When long-range variant effects such as 3D DNA contact are the quantity of interest
  — DeepFun's model has not considered long-range interactions
- When quantitative epigenomic signal levels are needed — DeepFun trains on binarized
  peaks and ignores quantitative information
- When the question is protein-coding variant consequence rather than non-coding regulatory
  impact
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0622
tools:
- name: DeepFun
  role: Pre-trained deep convolutional neural network (Basset-framework, Torch7/Python)
    whose SAD predictions score reference vs alternate allele effects across epigenomic
    tracks
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Underlying framework providing the SAD tutorial (sad.ipynb), basset_sad.py,
    and dependency installation for computing SNP Activity Difference profiles
  repo: https://github.com/davek44/Basset
- name: DeepSEA
  role: External deep learning framework whose SAD scores for the same variants and
    assays are used as comparison/reference scores for Pearson correlation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:021bf914b4e83d3aa0801038fff9a5be68e20d4a0fc2aecd5fc9be1e35ff7217
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  - DeepSEA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pretrained-cnn-variant-scoring

## Summary

Score non-coding SNVs for their functional impact on DNA accessibility, histone marks, and transcription factor binding (e.g. CTCF) by running DeepFun's pre-trained Basset-framework CNN (Torch7/Python) and computing SNP Activity Difference (SAD) profiles between reference and alternate alleles.

## When to use

Use when a set of non-coding SNP coordinates and alleles needs tissue- or cell type-specific functional impact scores across ENCODE/Roadmap epigenomic tracks, rather than a single pathogenicity score.

## When NOT to use

- When long-range variant effects such as 3D DNA contact are the quantity of interest — DeepFun's model has not considered long-range interactions
- When quantitative epigenomic signal levels are needed — DeepFun trains on binarized peaks and ignores quantitative information
- When the question is protein-coding variant consequence rather than non-coding regulatory impact

## Inputs

- Non-coding variant set as a BED/VCF-style file with chromosome, SNP coordinate, reference and alternate alleles
- Pre-trained DeepFun model file: DeepFun_best_A.th.gz (accessibility, histone marks, CTCF) or DeepFun_best_B.th.gz (transcription factor binding sites)
- Basset installation with Torch7, Python dependencies, pysam, and bedtools (BASSETDIR environment variable set)

## Outputs

- Per-variant SNP Activity Difference (SAD) score profiles across the model's epigenomic output tracks
- Per-assay averaged SAD scores (e.g. averaged across CTCF, DNase-seq, H3K4me3, H3K9ac tracks), e.g. written to deepfun_deepsea_sad_pcc.csv
- Pearson correlation coefficients comparing DeepFun vs DeepSEA per-assay SAD scores

## How to apply

Install Torch7 and the Basset Python dependencies following the Basset tutorials, then download the DeepFun pre-trained model DeepFun_best_A.th.gz (DNA accessibility sites, histone marks and Transcription factor CTCF binding sites; use DeepFun_best_B.th.gz for transcription factor binding sites) from http://bioinfo.uth.edu/deepfun/models/DeepFun_model/. Following the Basset SAD tutorial (sad.ipynb), run basset_sad.py to compute SNP Activity Difference profiles between reference and alternate alleles for each variant in the query set across the model's 7879 epigenomic output tracks (1548 DNase I, 1536 histone mark, 4795 TF binding profiles spanning 225 tissues/cell types). If scores must be aggregated per assay (e.g. CTCF, DNase-seq, H3K4me3, H3K9ac), average the SAD scores across all output tracks belonging to each assay and then compute Pearson correlations (numpy/scipy pearsonr) against scores from another model such as DeepSEA. For single variants, the DeepFun web server (https://bioinfo.uth.edu/deepfun/) can be used directly instead of local Torch7 execution.

## Related tools

- **DeepFun** (Pre-trained deep convolutional neural network (Basset-framework, Torch7/Python) whose SAD predictions score reference vs alternate allele effects across epigenomic tracks) — https://github.com/bsml320/DeepFun
- **Basset** (Underlying framework providing the SAD tutorial (sad.ipynb), basset_sad.py, and dependency installation for computing SNP Activity Difference profiles) — https://github.com/davek44/Basset
- **DeepSEA** (External deep learning framework whose SAD scores for the same variants and assays are used as comparison/reference scores for Pearson correlation)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(deepfun_avg_sad, deepsea_avg_sad)  # per-assay after averaging SAD tracks, e.g. CTCF PCC = 0.883
```

## Evaluation signals

- Sanity-check the per-assay aggregation: averaged SAD PCC values should be strong for CTCF (≈0.883, P = 2.2 × 10−16) and DNase-seq (≈0.751, P = 1.46 × 10−10), and moderate for H3K4me3 (≈0.490) and H3K9ac (≈0.451) when comparing DeepFun to DeepSEA
- Confirm the correct pre-trained model was downloaded and loaded (DeepFun_best_A.th.gz covers accessibility/histone/CTCF tracks; DeepFun_best_B.th.gz covers TF binding sites) — track counts should total 1548 + 1536 + 4795 = 7879
- Verify SAD computation ran for both reference and alternate alleles for every variant, with the tutorial script (basset_sad.py) completing without missing-sequence errors from the preprocessing (bedtools/pysam) step
- Check that Pearson correlations of per-assay averaged scores are computed with scipy pearsonr and exported to the four-row deepfun_deepsea_sad_pcc.csv
- Cross-validate a few variants against the DeepFun web server output to confirm local Torch7 predictions match

## Limitations

- DeepFun model has not considered long-range interactions, such as 3D DNA contact
- Peaks are treated as binary format during training, so quantitative epigenomic signal information is ignored and no normalization approach is yet established to integrate quantitative signals
- Performance of most CNN-based methods may strongly depend on initial convolution filters for captured motif information; local analysis requires installing Torch7 and Basset dependencies for large variant sets

## Evidence

- [methods] download pre-trained model (methods): "you'll need to download the pre-trained model from github here: dnacnn_best_A.zip (DNA accessibility sites, histone marks and Transcription factor CTCF binding sites)"
- [methods] SAD tutorial reference (methods): "More details can follow https://github.com/davek44/Basset/blob/master/tutorials/sad.ipynb."
- [abstract] feature space (abstract): "including 1548 DNase I accessibility, 1536 histone mark, and 4795 transcription factor binding profiles covering 225 tissues or cell types."
- [readme] variant annotation use case (readme): "The most useful capability of DeepFun right now is to annotate noncoding genomic variants with their influence on functional properties like accessibility."
- [discussion] 3D contact limitation (discussion): "DeepFun model has not considered long-range interactions, such as 3D DNA contact (20,37)"
- [discussion] binarized peaks (discussion): [evidence span withheld — anchor status: not_found]
