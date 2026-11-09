---
name: fasta-sequence-parsing
description: Use when the input is a FASTA file of DNA enhancer sequences (training
  or independent test set) that must be parsed per-record before feature extraction
  with dna2vec or motif PWM scanning.
when_to_use_negative:
- Input is already an extracted feature table in the 'feature' directory — skip parsing
  and run main.py directly
- Data is not in the expected FASTA layout (e.g. labels mixed into the file as in
  EPdata, where the first half of each file is labeled 0 and the second half 1) —
  reorganize first per README Step 0
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3474
tools:
- name: Enhancer-MDLF
  role: Framework whose feature-extraction scripts consume the parsed FASTA enhancer
    sequences
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: dna2vec
  role: Pre-trained DNA vectors in the 'dna2vec' folder looked up per k-mer after
    parsing each FASTA sequence
- name: Python 3.8
  role: Runtime for dna2vec_code.py and motif_find.py, which parse the FASTA input
- name: numpy
  role: Array operations for assembling per-sequence concatenated k-mer embeddings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:e8168a3090c272bc23c474fa555db770d5261d971b49ccfde2d8f7d933b20077
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
  repo_url: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
  tools:
  - Enhancer-MDLF
  - Python
  - Python 3.8
  - dna2vec
  - numpy
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# fasta-sequence-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reads enhancer FASTA sequence files (e.g. 'data/train/NHLF.fasta' from the Enhancer-MDLF data folder) into memory so per-sequence k-mer embedding and motif features can be extracted for the deep learning model.

## When to use

Use when the input is a FASTA file of DNA enhancer sequences (training or independent test set) that must be parsed per-record before feature extraction with dna2vec or motif PWM scanning.

## When NOT to use

- Input is already an extracted feature table in the 'feature' directory — skip parsing and run main.py directly
- Data is not in the expected FASTA layout (e.g. labels mixed into the file as in EPdata, where the first half of each file is labeled 0 and the second half 1) — reorganize first per README Step 0

## Inputs

- FASTA file of enhancer sequences, e.g. 'data/train/NHLF.fasta' or 'data/test/NHLF.fasta' (same format for EPdata and generic Liu et al. datasets)
- Cell line name (e.g. NHLF) and set type ('train' or 'test') used to select/label the parsed records

## Outputs

- In-memory per-record DNA sequences ready for k-mer (k=3) embedding lookup and motif PWM scanning
- Downstream feature vectors saved in the 'feature' directory (dna2vec-based 10,000-dim features and motif-based features)

## How to apply

Load the enhancer FASTA file using the naming convention from the data folder, e.g. 'data/train/${cell line name}.fasta' for the training set and 'data/test/${cell line name}.fasta' for the test set, keeping the cell line name (e.g. NHLF) and set type aligned with the label files (${cell_line}_y_train.txt / ${cell_line}_y_test.txt). Parse each record so that per-sequence nucleotide strings are available downstream: dna2vec_code.py slides a k=3 window with stride 1 over each sequence and looks up each k-mer's pre-trained 100-dim vector from the 'dna2vec' folder, while motif_find.py scans each sequence against HOCOMOCO Human v11 PWMs with p-value threshold scores. Repeat the parsing separately for train and test sets by passing --input_file, --cell_line, and --set; feature outputs are written to the 'feature' directory. Correct application is confirmed when every parsed sequence yields the fixed 10,000-dimensional dna2vec input expected by Enhancer-MDLF.

## Related tools

- **Enhancer-MDLF** (Framework whose feature-extraction scripts consume the parsed FASTA enhancer sequences) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **dna2vec** (Pre-trained DNA vectors in the 'dna2vec' folder looked up per k-mer after parsing each FASTA sequence)
- **Python 3.8** (Runtime for dna2vec_code.py and motif_find.py, which parse the FASTA input)
- **numpy** (Array operations for assembling per-sequence concatenated k-mer embeddings)

## Examples

```
python dna2vec_code.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train
```

## Evaluation signals

- Each parsed sequence yields a fixed 10,000-dimensional dna2vec feature vector matching the Enhancer-MDLF network input size
- Output feature files exist in the 'feature' directory after running dna2vec_code.py and motif_find.py
- Sequence counts per cell line/set match the corresponding label file (${cell_line}_y_train.txt / ${cell_line}_y_test.txt) row counts
- Feature extraction succeeds for both --set train and --set test runs on the same cell line

## Limitations

- Users supplying their own datasets must organize them exactly in the README Step 0 FASTA/label format or parsing breaks
- Parsing quality depends on the eight-cell-line data folder layout; EPdata files embed labels by position rather than separate label files

## Evidence
