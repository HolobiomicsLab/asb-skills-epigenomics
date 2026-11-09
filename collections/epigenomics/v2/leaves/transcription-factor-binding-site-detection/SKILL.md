---
name: transcription-factor-binding-site-detection
description: Use when converting enhancer FASTA sequences (e.g. data/train/NHLF.fasta)
  into per-motif TFBS count/frequency features for a deep learning enhancer predictor,
  using HOCOMOCO v11 PWMs with p-value thresholds.
when_to_use_negative:
- Input is already an extracted feature table (motif frequency vectors); re-running
  motif extraction would duplicate Step 2
- The goal is dna2vec k-mer word vector features rather than PWM-based TFBS motif
  features; use dna2vec_code.py instead
- 'Sequences are non-human enhancers: the PWMs and p-value thresholds come from the
  HOCOMOCO Human v11 database, so TFBS counts for other species are not supported
  by the provided motif files'
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: motif_find.py
  role: Script that performs the sliding-window PWM matching and writes the 401-dim
    motif frequency feature files for a given cell line and train/test set
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: HOCOMOCO Human v11 database
  role: Source of the transcription factor binding motif position weight matrices
    (PWMs) and their p-value threshold scores used as matching thresholds
- name: Python 3.8
  role: Runtime environment in which motif_find.py is executed (conda env with pip
    install -r requirements.txt)
- name: numpy
  role: Numeric library dependency used by the motif feature extraction script
- name: Enhancer-MDLF
  role: Downstream multi-input deep learning framework that consumes the extracted
    motif frequency features as one of its inputs (run via main.py --cell_line)
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:cfb16879a17a1aadc536f03bc47bc8d13987829458089da330c320b5281ce7d1
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
  - HOCOMOCO Human v11 database
  - Python
  - Python 3.8
  - motif_find.py
  - numpy
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# transcription-factor-binding-site-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes cell-line enhancer DNA sequences into fixed-length motif frequency feature vectors by matching sequences against transcription factor binding motif PWMs from the HOCOMOCO Human v11 database. This motif feature input is one of the two feature streams consumed by the Enhancer-MDLF multi-input deep learning framework for cell-specific enhancer prediction.

## When to use

Use when converting enhancer FASTA sequences (e.g. data/train/NHLF.fasta) into per-motif TFBS count/frequency features for a deep learning enhancer predictor, using HOCOMOCO v11 PWMs with p-value thresholds.

## When NOT to use

- Input is already an extracted feature table (motif frequency vectors); re-running motif extraction would duplicate Step 2
- The goal is dna2vec k-mer word vector features rather than PWM-based TFBS motif features; use dna2vec_code.py instead
- Sequences are non-human enhancers: the PWMs and p-value thresholds come from the HOCOMOCO Human v11 database, so TFBS counts for other species are not supported by the provided motif files

## Inputs

- Enhancer training set FASTA sequences for a cell line (e.g. data/train/NHLF.fasta)
- Enhancer independent test set FASTA sequences for a cell line (e.g. data/test/NHLF.fasta)
- Position weight matrices (PWMs) of transcription factor binding motifs from the HOCOMOCO Human v11 database (folder 'motif')
- P-value threshold scores for each motif from the HOCOMOCO Human v11 database (folder 'motif')
- Command-line arguments: --input_file, --cell_line, --set (train or test)

## Outputs

- One motif frequency feature vector per input sequence, saved in the 'feature' directory
- 401-dimensional per-sequence motif vectors (one dimension per motif), counts normalized to sequence length
- Separate feature files for the requested cell line and set (train/test)

## How to apply

Load the enhancer training and test FASTA sequences for the target cell line (e.g. data/train/NHLF.fasta) and load the position weight matrices (PWMs) of transcription factor binding motifs together with their p-value threshold scores from the HOCOMOCO Human v11 database files in the 'motif' folder. For each sequence, slide each PWM along the sequence, score every window, and retain windows whose score passes the motif's p-value threshold to count TFBS per motif. Assemble the per-motif TFBS counts into a fixed-length 401-dimensional motif count vector (one dimension per motif), then convert counts to motif frequencies by dividing each count by the sequence length. Execute the extraction with motif_find.py for both the train and test sets (e.g. python motif_find.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train), writing outputs to the 'feature' directory alongside the dna2vec features produced by dna2vec_code.py.

## Related tools

- **motif_find.py** (Script that performs the sliding-window PWM matching and writes the 401-dim motif frequency feature files for a given cell line and train/test set) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **HOCOMOCO Human v11 database** (Source of the transcription factor binding motif position weight matrices (PWMs) and their p-value threshold scores used as matching thresholds)
- **Python 3.8** (Runtime environment in which motif_find.py is executed (conda env with pip install -r requirements.txt))
- **numpy** (Numeric library dependency used by the motif feature extraction script)
- **Enhancer-MDLF** (Downstream multi-input deep learning framework that consumes the extracted motif frequency features as one of its inputs (run via main.py --cell_line)) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF

## Examples

```
python motif_find.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train
```

## Evaluation signals

- motif_find.py completes without error for both the train and test sets of the requested cell line
- The 'feature' directory contains a motif feature file for each requested cell line/set
- Each input sequence is represented by a motif frequency vector of the expected 401-motif dimensionality
- Vector values are normalized: each per-motif TFBS count divided by the sequence length (motif frequencies, not raw counts)
- Downstream main.py training/prediction accepts the feature files without shape errors (e.g. run for NHLF)

## Limitations

- Motif PWMs and p-value thresholds are specific to the HOCOMOCO Human v11 database, so detected TFBS reflect human transcription factor binding models
- Data must be organized in the README's directory/format conventions (data/train/${cell line}.fasta, data/test/${cell line}.fasta) or the extraction will not find its inputs
- Motif features must be extracted separately for train and test sets (--set train / --set test); skipping either yields an incomplete feature set for training and prediction

## Evidence
