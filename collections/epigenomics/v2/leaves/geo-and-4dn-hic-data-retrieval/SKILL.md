---
name: geo-and-4dn-hic-data-retrieval
description: Use when a transfer-learning or cross-cell-type loop-calling study requires
  source/target human Hi-C contact maps, and the target cell line (e.g. K562, HeLa-S3)
  data must be obtained from GEO or 4DN accessions before GILoop inference.
when_to_use_negative:
- The analysis targets single-cell Hi-C (scHi-C) data — GILoop 'only focuses on bulk
  Hi-C' and cannot handle extremely sparse scHi-C matrices
- The target cell line's sequencing depth matches none of the pre-trained models in
  the GILoop repository, which the authors flag as a 'weak usability' scenario requiring
  retraining
- The task needs loop calling directly on .hic files with an alternative caller (e.g.
  SIP on Juicer .hic files) rather than retrieving data for GILoop
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: GILoop
  role: Downstream dual-branch loop caller whose required input format (intra-chromosomal
    interaction matrices and graph structures, .cool or Juicer dump) dictates how
    the retrieved Hi-C data must be preprocessed
  repo: github.com/fzbio/GILoop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:70ec149f7ed093589d33be52121eb2f40455e4c09582d7ab6cf8e5219e54d2b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# geo-and-4dn-hic-data-retrieval

## Summary

Retrieves public bulk Hi-C datasets for cross-cell-line loop-calling experiments — K562 Hi-C from GEO (GSE63525) and HeLa-S3 Hi-C from 4DN (4DNESCMX7L58) — to serve as transfer targets for GILoop models trained on GM12878.

## When to use

Use when a transfer-learning or cross-cell-type loop-calling study requires source/target human Hi-C contact maps, and the target cell line (e.g. K562, HeLa-S3) data must be obtained from GEO or 4DN accessions before GILoop inference.

## When NOT to use

- The analysis targets single-cell Hi-C (scHi-C) data — GILoop 'only focuses on bulk Hi-C' and cannot handle extremely sparse scHi-C matrices
- The target cell line's sequencing depth matches none of the pre-trained models in the GILoop repository, which the authors flag as a 'weak usability' scenario requiring retraining
- The task needs loop calling directly on .hic files with an alternative caller (e.g. SIP on Juicer .hic files) rather than retrieving data for GILoop

## Inputs

- GEO accession GSE63525 (K562 Hi-C raw/processed data)
- 4DN accession 4DNESCMX7L58 (HeLa-S3 Hi-C data)
- GM12878 Hi-C source dataset used to train GILoop models
- GILoop pre-trained model checkpoints (e.g. 20%- and 50%-downsampled GM12878 models) from the GILoop_assets package

## Outputs

- Intra-chromosomal interaction matrices and graph structures formatted for GILoop's dual-branch (graph/image) input
- KR-normalized O/E matrices or .cool files ready for GILoop patching and inference

## How to apply

Download the K562 Hi-C dataset from GEO accession GSE63525 and the HeLa-S3 Hi-C dataset from the 4DN repository accession 4DNESCMX7L58, matching the cell lines used in GILoop's Figure 6 transfer-learning case studies. Also obtain the GM12878 source data and the corresponding pre-trained GILoop checkpoints (e.g. the 20%-downsampled model for K562 and the 50% model for HeLa-S3). Preprocess each contact map into intra-chromosomal interaction matrices and graph structures in GILoop's dual-branch input format; GILoop accepts either .cool files (via demo.py, with sequencing-depth alignment, normalization, and expected-vector calculation) or Juicer dump text files (via demo_from_processed.py, requiring KR-normalized O/E matrices). Ensure source and target data are of a similar number of reads when using the Juicer-dump path, mirroring the downsampling scheme (e.g. 20%, 50%) applied to the GM12878 training data.

## Related tools

- **GILoop** (Downstream dual-branch loop caller whose required input format (intra-chromosomal interaction matrices and graph structures, .cool or Juicer dump) dictates how the retrieved Hi-C data must be preprocessed) — github.com/fzbio/GILoop

## Evaluation signals

- The K562 download resolves to GEO accession GSE63525 and the HeLa-S3 download resolves to 4DN accession 4DNESCMX7L58, matching the transfer case-study cell lines
- Retrieved contact maps convert into valid intra-chromosomal interaction matrices with graph structures accepted by GILoop's dual-branch pipeline
- When using demo_from_processed.py, source and target text files are KR-normalized O/E matrices of similar read counts (source GM12878 downsampled to 20% for K562, 50% for HeLa-S3)
- The 'models' and 'data' directories from GILoop_assets have been downloaded and replaced locally before running inference
- Retrieval supports downstream transfer validation: target PR curves should lie near-identically to the GM12878 source baseline (minimal performance decay)

## Limitations

- GILoop's preprocessing path is slow — the README notes 'The preprocessing overhead could be heavy and could lead to a large running time'
- The Juicer-dump input path applies no preprocessing: 'please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads'
- Usability is weak if the retrieved cell line's sequencing depth differs from any pre-trained model provided in the repository, per the article discussion

## Evidence

- [other] Download the K562 Hi-C dataset from GEO accession GSE63525 and the HeLa-S3 Hi-C dataset from the 4DN repository accession 4DNESCMX7L58.: [evidence span withheld — anchor status: not_found]
- [abstract] Transfer-learning case studies across cell lines: "along with multiple transfer-learning case studies, we demonstrate that GILoop can accurately model the organizational and functional patterns of CTCF-mediated looping across different cell lines"
- [readme] Input formats supported by GILoop: "You can either use `.cool` files as input, or use the output format of Juicer dump as input."
- [readme] Assets must be downloaded and installed locally: "please download `models` and `data` from [GILoop_assets](https://portland-my.sharepoint.com/:f:/g/personal/fuzhowang2-c_my_cityu_edu_hk/EpsC_y58ARNInLGjwy4yc44BNs2fKzCXNFVLUxrsrtHO2A?e=83KzE4)"
