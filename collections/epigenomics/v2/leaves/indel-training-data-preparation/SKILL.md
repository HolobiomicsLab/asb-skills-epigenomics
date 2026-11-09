---
name: indel-training-data-preparation
description: Use when building a supervised nc-sindel classifier for tissue-specific
  regulatory indels and you need to convert indel position/ref/alt files into a CADD
  + DanQ multi-omics feature table for model training.
when_to_use_negative:
- You only need precomputed tissue-specific functional scores for known nc-sindels
  — use R/TIVAN-indel.R with the precomputed 44-GTEx-tissue score files instead of
  rebuilding features.
- Your variants are SNVs rather than small indels — the CADD.indel.rda annotations
  and the sequence-injecting prepare.R logic are specific to indels.
- Input is already a feature table (annot.csv plus seq_features.h5 exist) — skip preparation
  and go directly to train.py or predict.py.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: prepare.R
  role: R script (BSgenome.Hsapiens.UCSC.hg19, data.table) that extracts CADD annotations
    from CADD.indel.rda and writes reference/alternative fasta sequences for query
    indels
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: src/prepare.py
  role: Python script calling the pretrained DanQ model from checkpoint/ to predict
    919 multi-omics features from reference/alternative sequences into seq_features.h5
  repo: https://github.com/lichen-lab/TIVAN-indel
- name: h5py
  role: reads/writes seq_features.h5 multi-omics feature file
- name: BSgenome.Hsapiens.UCSC.hg19
  role: reference genome used to retrieve reference and alternative indel sequences
    in prepare.R
- name: CADD.indel.rda
  role: source rda file of CADD annotations for all 1000 Genomes indels queried by
    prepare.R
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad060
    title: 'TIVAN-indel: a computational framework for annotating and predicting non-coding
      regulatory small insertions and deletions'
  source_content_hash: sha256:11105b76d75fb55f9b755f5efbe9b8c8e5df69ed0da95eb4496f33708f5c3201
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/TIVAN-indel
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - CADD.indel.rda
  - Python
  - R
  - TIVAN-indel
  - TensorFlow
  - data.table
  - h5py
  - numpy
  - pandas
  - prepare.R
  - scikit-learn
  - src/prepare.py
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# indel-training-data-preparation

## Summary

Generates the labeled training features for TIVAN-indel by combining 45 CADD annotations (via an R rda file covering 1000 Genomes indels) with 919 multi-omics features predicted from reference/alternative indel sequences using the pretrained DanQ model, producing annot.csv and seq_features.h5 in a target folder.

## When to use

Use when building a supervised nc-sindel classifier for tissue-specific regulatory indels and you need to convert indel position/ref/alt files into a CADD + DanQ multi-omics feature table for model training.

## When NOT to use

- You only need precomputed tissue-specific functional scores for known nc-sindels — use R/TIVAN-indel.R with the precomputed 44-GTEx-tissue score files instead of rebuilding features.
- Your variants are SNVs rather than small indels — the CADD.indel.rda annotations and the sequence-injecting prepare.R logic are specific to indels.
- Input is already a feature table (annot.csv plus seq_features.h5 exist) — skip preparation and go directly to train.py or predict.py.

## Inputs

- snp.pos.txt: positive-set indels, one per line as 'chr start end refallele altallele'
- snp.neg.txt: negative-set indels in the same format
- CADD.indel.rda: R rda file with CADD annotations for all indels in 1000 Genomes Project
- Pretrained DanQ model weights in checkpoint/ (model code in models/)

## Outputs

- annot.csv: 45 CADD annotations for both positive and negative indels
- snp.pos.ref.fasta / snp.pos.alt.fasta: reference and alternative genomic sequences for positive indels
- snp.neg.ref.fasta / snp.neg.alt.fasta: reference and alternative genomic sequences for negative indels
- seq_features.h5: h5 file of 919 DanQ-predicted multi-omics features per indel

## How to apply

Start from positive and negative indel files (snp.pos.txt, snp.neg.txt) with 'chr start end refallele altallele' columns and the CADD.indel.rda file containing CADD annotations for all 1000 Genomes indels. Run the R script prepare.R with these four arguments and an output folder (e.g. 'train'); it writes annot.csv (45 CADD annotations for positive and negative indels) plus snp.{pos,neg}.{ref,alt}.fasta sequences using BSgenome.Hsapiens.UCSC.hg19. Then run src/prepare.py --path <folder>, which loads the pretrained DanQ model from checkpoint/ (model code in models/) and predicts 919 multi-omics features from the reference/alternative sequences, saving seq_features.h5 in that folder. The combined 45 + 919 feature matrix feeds train.py; repeat the same two steps (predict.R then prepare.py) on test indels before prediction. Rationale: tissue-specific regulatory prediction benefits from pairing generic functional annotations with sequence-derived epigenomic features.

## Related tools

- **prepare.R** (R script (BSgenome.Hsapiens.UCSC.hg19, data.table) that extracts CADD annotations from CADD.indel.rda and writes reference/alternative fasta sequences for query indels) — https://github.com/lichen-lab/TIVAN-indel
- **src/prepare.py** (Python script calling the pretrained DanQ model from checkpoint/ to predict 919 multi-omics features from reference/alternative sequences into seq_features.h5) — https://github.com/lichen-lab/TIVAN-indel
- **h5py** (reads/writes seq_features.h5 multi-omics feature file)
- **BSgenome.Hsapiens.UCSC.hg19** (reference genome used to retrieve reference and alternative indel sequences in prepare.R)
- **CADD.indel.rda** (source rda file of CADD annotations for all 1000 Genomes indels queried by prepare.R)

## Examples

```
R --slave --args --no-save CADD.indel.rda train/snp.pos.txt train/snp.neg.txt train < prepare.R && python prepare.py --path ../train
```

## Evaluation signals

- annot.csv exists in the output folder and contains CADD annotation rows for both positive and negative indels
- Exactly five files are produced in the target folder: annot.csv and four fasta files (ref/alt for pos/neg sets)
- seq_features.h5 is generated in the target folder after running src/prepare.py --path <folder>
- The feature count matches the model expectation: 45 CADD annotations plus 919 DanQ multi-omics features, verifiable via train.py feature dimensions
- Downstream check: training on the assembled features and evaluating on the 15 DICE immune cell types should reach median AUROC ~0.973 and AUPRC ~0.978 per Figure 5

## Limitations

- CADD annotations must be provided as the precomputed CADD.indel.rda (1000 Genomes indels) downloaded from the repository's Google Drive link; indels absent from that file cannot be annotated this way.
- The 919 multi-omics features rely on the pretrained DanQ weights in checkpoint/ (hg19/BSgenome.Hsapiens.UCSC.hg19); retraining the DanQ model requires DeepSEA data (http://deepsea.princeton.edu/help/).
- Files must follow the 'chr start end refallele altallele' format or the R feature-extraction step will not locate indels.

## Evidence

- [readme] R script ``prepare.R" will generated (1) CADD annotations for query sindels and (2) reference genomic sequence and alterntive genomic sequence with sindels: "R script `prepare.R` will generated (1) CADD annotations for query sindels and (2) reference genomic sequence and alterntive genomic sequence with sindels."
- [methods] File `src/prepare.py` calls DanQ model to predict 919 multi-omics features: "File `src/prepare.py` calls DanQ model to predict 919 multi-omics features"
- [readme] The trained weights of DanQ model are provided in the `checkpoint/` directory, while the model code is present in `models/` directory.: "The trained weights of DanQ model are provided in the `checkpoint/` directory, while the model code is present in `models/` directory."
- [readme] It uses a compilation of both 45 CADD annotations and 919 multi-omics features as generated above.: "It uses a compilation of both 45 CADD annotations and 919 multi-omics features as generated above."
