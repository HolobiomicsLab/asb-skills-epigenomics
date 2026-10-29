---
name: p-value-threshold-based-motif-filtering
description: Use when converting cell-line enhancer FASTA sequences (e.g. data/train/NHLF.fasta)
  into numeric motif features for enhancer prediction, requiring PWM sliding-window
  scanning with p-value threshold filtering.
when_to_use_negative:
- The input is already a numeric feature table or motif count matrix rather than raw
  DNA FASTA sequences
- You need dna2vec k-mer word-vector features instead of TFBS motif features (use
  dna2vec_code.py workflow instead)
- The organism of interest is not covered by the HOCOMOCO Human v11 motif collection
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3510
tools:
- name: motif_find.py
  role: Script that performs the sliding-scale PWM matching, p-value threshold filtering,
    and motif frequency vector extraction
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: HOCOMOCO Human v11 database
  role: Source of the motif PWMs and the p-value threshold scores used for filtering
    TFBS matches
- name: numpy
  role: Numerical computation dependency for assembling and normalizing the 401-dimensional
    motif frequency vectors
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:3cf513e0ba7e09a0f11a45e4f969629c8eb2c116f50246ed0bf52332ad2a6a63
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
  - HOCOMOCO Human v11 database
  - Python
  - motif_find.py
  - numpy
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3510
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# p-value-threshold-based-motif-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes DNA enhancer sequences into transcription factor binding site (TFBS) motif frequency features by sliding HOCOMOCO Human v11 PWMs along each sequence and retaining only windows whose score passes the motif's p-value threshold. This produces a fixed-length per-sequence motif vector used as one input to Enhancer-MDLF.

## When to use

Use when converting cell-line enhancer FASTA sequences (e.g. data/train/NHLF.fasta) into numeric motif features for enhancer prediction, requiring PWM sliding-window scanning with p-value threshold filtering.

## When NOT to use

- The input is already a numeric feature table or motif count matrix rather than raw DNA FASTA sequences
- You need dna2vec k-mer word-vector features instead of TFBS motif features (use dna2vec_code.py workflow instead)
- The organism of interest is not covered by the HOCOMOCO Human v11 motif collection

## Inputs

- Enhancer FASTA sequences for a cell line train/test set (e.g. data/train/NHLF.fasta, data/test/NHLF.fasta)
- Position weight matrices (PWMs) of transcription factor binding motifs from the 'motif' folder
- p-value threshold scores for each motif from the HOCOMOCO Human v11 database

## Outputs

- A motif frequency vector per input sequence (401 dimensions, one per motif), normalized by sequence length
- Motif feature files written to the 'feature' directory for the requested cell line and set (train/test)

## How to apply

Load the enhancer train/test FASTA for the target cell line and load the motif PWMs with their p-value threshold scores from the 'motif' folder (HOCOMOCO Human v11 database). For each sequence, slide each of the 401 PWMs along the sequence, score every window, and retain only windows scoring above the motif's p-value threshold, counting one TFBS per retained window per motif. Assemble the per-motif TFBS counts into a fixed-length 401-dimensional count vector, then divide each count by the sequence length to obtain motif frequencies. Run the extraction via motif_find.py for both train and test sets, saving one normalized frequency vector per sequence in the 'feature' directory. The p-value threshold is the key decision point: it trades off motif match specificity (strict thresholds miss real sites) against sensitivity (loose thresholds inflate counts), which directly affects downstream enhancer prediction.

## Related tools

- **motif_find.py** (Script that performs the sliding-scale PWM matching, p-value threshold filtering, and motif frequency vector extraction) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **HOCOMOCO Human v11 database** (Source of the motif PWMs and the p-value threshold scores used for filtering TFBS matches)
- **numpy** (Numerical computation dependency for assembling and normalizing the 401-dimensional motif frequency vectors)

## Examples

```
python motif_find.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train
```

## Evaluation signals

- The motif_find.py script completes without error for both train and test sets of the target cell line
- The 'feature' directory contains a motif feature file for each requested cell line/set
- Each sequence is represented by a motif frequency vector of the expected 401-motif dimensionality
- Vector values are normalized to the sequence length (frequencies, not raw counts)
- The extracted motif features feed Enhancer-MDLF (main.py --cell_line ${cell_line}) without shape or dimensionality errors

## Limitations

- Motif scan quality depends on the HOCOMOCO Human v11 PWMs and their p-value thresholds; cell-specific TFBS motifs outside this collection cannot be captured
- Feature extraction must be run separately for each cell line and each set (train/test), as encoding is cell-line-specific
- The output is a fixed 401-dimension vector tied to the motifs bundled in the 'motif' folder; changing the motif set changes the feature dimensionality

## Evidence
