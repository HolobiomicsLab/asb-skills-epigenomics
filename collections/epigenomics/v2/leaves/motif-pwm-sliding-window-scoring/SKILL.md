---
name: motif-pwm-sliding-window-scoring
description: Use when raw DNA sequences (e.g. FASTA enhancer sequences per cell line)
  must be converted into per-motif TFBS count/frequency features for downstream machine
  learning, and a curated PWM library with p-value thresholds is available.
when_to_use_negative:
- The input is already a numeric feature table (e.g. precomputed motif frequencies)
  — re-running extraction would be redundant.
- The task is k-mer/embedding-based sequence encoding — use dna2vec_code.py for dna2vec-based
  features instead.
- Non-human or non-mouse sequences where HOCOMOCO Human v11 PWMs and their p-value
  thresholds are not appropriate.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3510
- http://edamontology.org/topic_0749
tools:
- name: motif_find.py
  role: Script that performs the PWM sliding-window matching and writes motif frequency
    features
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: HOCOMOCO Human v11 database
  role: Source of motif position weight matrices (PWMs) and p-value threshold scores
    used for TFBS matching
- name: numpy
  role: Numerical dependency used by the motif feature extraction script
- name: Enhancer-MDLF
  role: Downstream multi-input deep learning framework that consumes the motif frequency
    features
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:d9329d993db5c90c93b01c502f809bd50a7cfbc0db8b8067041280250b02b1c9
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
  - motif_find.py
  - numpy
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3510
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# motif-pwm-sliding-window-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes DNA enhancer sequences into fixed-length motif frequency vectors by sliding HOCOMOCO Human v11 position weight matrices (PWMs) along each sequence, counting TFBS windows that pass each motif's p-value threshold, and normalizing counts by sequence length. This yields 401-dimensional motif features for cell-specific enhancer prediction with Enhancer-MDLF.

## When to use

Use when raw DNA sequences (e.g. FASTA enhancer sequences per cell line) must be converted into per-motif TFBS count/frequency features for downstream machine learning, and a curated PWM library with p-value thresholds is available.

## When NOT to use

- The input is already a numeric feature table (e.g. precomputed motif frequencies) — re-running extraction would be redundant.
- The task is k-mer/embedding-based sequence encoding — use dna2vec_code.py for dna2vec-based features instead.
- Non-human or non-mouse sequences where HOCOMOCO Human v11 PWMs and their p-value thresholds are not appropriate.

## Inputs

- Enhancer FASTA sequences for a cell line (e.g. data/train/NHLF.fasta, data/test/NHLF.fasta)
- HOCOMOCO Human v11 PWM files and p-value threshold scores from the 'motif' folder
- CLI parameters: --input_file, --cell_line, --set (train or test)

## Outputs

- Motif feature files in the 'feature' directory, one motif frequency vector per input sequence
- Fixed-length 401-dimensional per-sequence motif frequency vectors (counts normalized by sequence length)

## How to apply

Load the enhancer training and test FASTA sequences for the target cell line (e.g. data/train/NHLF.fasta and data/test/NHLF.fasta) and load the motif PWMs together with their p-value threshold scores from the HOCOMOCO Human v11 database files in the 'motif' folder. For each sequence, slide each PWM along the sequence, score every window, and retain windows whose score passes the motif's p-value threshold to count transcription factor binding sites per motif. Assemble the per-motif TFBS counts into a fixed-length 401-dimensional motif count vector, then convert counts to motif frequencies by dividing each count by the sequence length. Execute the extraction via motif_find.py for both train and test sets and verify the outputs in the 'feature' directory. This motif-frequency encoding is one of the two feature streams (alongside dna2vec features) feeding the Enhancer-MDLF multi-input deep learning framework.

## Related tools

- **motif_find.py** (Script that performs the PWM sliding-window matching and writes motif frequency features) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **HOCOMOCO Human v11 database** (Source of motif position weight matrices (PWMs) and p-value threshold scores used for TFBS matching)
- **numpy** (Numerical dependency used by the motif feature extraction script)
- **Enhancer-MDLF** (Downstream multi-input deep learning framework that consumes the motif frequency features) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF

## Examples

```
python motif_find.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train
```

## Evaluation signals

- The script completes without error for both train and test sets (e.g. NHLF).
- The 'feature' directory contains a motif feature file for each requested cell line/set.
- Each sequence is represented by a motif frequency vector of the expected 401-motif dimensionality.
- Vector values are normalized to the sequence length (counts divided by sequence length), so features are frequencies rather than raw counts.
- Downstream Enhancer-MDLF training (python main.py --cell_line ${cell_line}) can consume the feature files without shape errors.

## Limitations

- Motif p-value thresholds come from HOCOMOCO Human v11; performance depends on that database's coverage and calibration.
- The output is a fixed 401-motif dimensionality, so motifs outside that panel are not captured.
- Features are extracted per cell line and per set (train/test); data must be organized in the expected 'data/train/${cell line name}.fasta' / 'data/test/${cell line name}.fasta' format before extraction.
- The extracted motif features must be paired with dna2vec features for the full Enhancer-MDLF multi-input model; motif features alone are only one input stream.

## Evidence
