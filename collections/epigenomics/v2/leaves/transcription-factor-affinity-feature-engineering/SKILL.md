---
name: transcription-factor-affinity-feature-engineering
description: Use when raw DNA binding sequences from uPBM, HT-SELEX or gcPBM must
  be converted into structural/mechanical feature matrices to train or apply a transcription-factor
  affinity regressor.
when_to_use_negative:
- Input is already a feature table or preprocessed data file (e.g. Gata4_training.txt
  from the test_data folder) ready for the regressor — skip to {method}_regressor.py
- The goal is de novo motif discovery rather than affinity prediction — use MEME suite,
  BEESEM, or BindSpace instead
- Sequences contain non-standard nucleobases or epigenetic variants beyond the tetramer
  parameter database, where lookups will fail
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Package providing the tetramer feature-extraction code and the full database
    of indirect/direct readout parameters; trained via random forest regressor
  repo: https://github.com/Jalbiti/DNAffinity
- name: MEME suite
  role: Used to trim uPBM sequences before feature extraction, reducing sequence variability
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:a77db1a92bafa8f7ea2e8b34e01225f6a693a548335860873e98630030f15d33
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Jalbiti/DNAffinity
  repo_url: https://github.com/Jalbiti/DNAffinity
  tools:
  - DNAffinity
  - MEME suite
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# transcription-factor-affinity-feature-engineering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Engineer DNA sequence features for the DNAffinity random-forest affinity predictor: overlapping tetramer counts plus indirect-readout (MD-derived equilibrium geometry and flexibility) and direct-readout (electrostatic) base-pair-step parameters, feeding {method}_regressor.py.

## When to use

Use when raw DNA binding sequences from uPBM, HT-SELEX or gcPBM must be converted into structural/mechanical feature matrices to train or apply a transcription-factor affinity regressor.

## When NOT to use

- Input is already a feature table or preprocessed data file (e.g. Gata4_training.txt from the test_data folder) ready for the regressor — skip to {method}_regressor.py
- The goal is de novo motif discovery rather than affinity prediction — use MEME suite, BEESEM, or BindSpace instead
- Sequences contain non-standard nucleobases or epigenetic variants beyond the tetramer parameter database, where lookups will fail

## Inputs

- proteins/{protein}/{protein}_training.txt (uPBM training sequences, post-MEME trimming)
- proteins/{protein}/{cycle}.txt (HT-SELEX cycle sequences)
- proteins/{protein}/{protein}_{concentration}.txt and proteins/{protein}/_freq_matrix_6.txt (gcPBM)
- MD-derived tetramer tables of indirect-readout parameters (BigNASim AVG equilibrium and DIAG stiffness values)

## Outputs

- Per-sequence tetramer-count composition feature vector
- Per-tetramer indirect-readout features (6 helical coordinates: shift, slide, rise, tilt, roll, twist with AVG and DIAG stiffness entries)
- Per-tetramer direct-readout electrostatic feature vector (5 column-summed values)
- Assembled feature matrix consumed by {method}_regressor.py

## How to apply

First load the required sequence files — proteins/{protein}/{protein}_training.txt for uPBM, proteins/{protein}/{cycle}.txt for HT-SELEX, or proteins/{protein}/{protein}_{concentration}.txt plus proteins/{protein}/_freq_matrix_6.txt for gcPBM — checking formatting against the shipped templates. Slide a 4-mer window across each sequence and count overlapping tetranucleotide occurrences to build the composition feature. For indirect readout, look up each tetramer in the MD-derived tetramer tables (BigNASim), taking the AVG equilibrium values and DIAG stiffness diagonal entries for the six helical coordinates (shift, slide, rise, tilt, roll, twist). For direct readout, assign the integer electrostatic codes for acceptor, donor, and hydrophobic chemical groups per base step; the paper's worked example for 'AACT' sums four base codes to give [-2, 2, -2, -2, 1]. Assemble the three feature blocks and pass them to {method}_regressor.py with the protein name as the 1st argument and the cycle number or concentration as the 2nd argument where applicable.

## Related tools

- **DNAffinity** (Package providing the tetramer feature-extraction code and the full database of indirect/direct readout parameters; trained via random forest regressor) — https://github.com/Jalbiti/DNAffinity
- **MEME suite** (Used to trim uPBM sequences before feature extraction, reducing sequence variability)

## Examples

```
python upbm_regressor.py Gata4; python selex_regressor.py Gata4 4
```

## Evaluation signals

- Reproduce the paper's worked 'AACT' example: the direct-readout electrostatic feature vector must equal [-2, 2, -2, -2, 1]
- Each tetramer's indirect-readout block contains exactly 6 helical coordinates (shift, slide, rise, tilt, roll, twist) with AVG equilibrium and DIAG stiffness diagonal entries
- Feature matrix dimensions match the number of overlapping tetramer windows across the input sequences
- Regressor runs accept the correct argument pattern: protein name as 1st argument, cycle number (selex) or concentration (gcpbm) as 2nd argument
- Downstream regressor performance should be consistent with published baselines (e.g. R2 = 0.93 ± 0.02 on gcPBM, R2 = 0.69 ± 0.17 on uPBM)

## Limitations

- The transcription factor binding site concept is not uniquely defined — it depends on the intrinsically noisy and low-resolution experimental technique used to detect it, preventing a universal predictor
- Extrapolating in vitro predictions to in vivo settings is complicated by factors beyond intrinsic transcription factor affinity; DNAffinity predicts many potential binding sites with no experimental evidence of in vivo binding
- Input files must exactly match the shipped template formatting; the README instructs users to 'compare formatting with the templates' before running

## Evidence
