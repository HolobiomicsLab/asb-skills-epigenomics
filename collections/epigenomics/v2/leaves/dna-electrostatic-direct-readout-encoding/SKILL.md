---
name: dna-electrostatic-direct-readout-encoding
description: Use when building DNAffinity feature matrices from DNA sequences and
  the direct-readout electrostatic block must be derived per tetramer, or when validating
  the encoding against the AACT worked example.
when_to_use_negative:
- Input is already a DNAffinity feature matrix or a precomputed tetramer feature table
  — the encoding step is already done
- The analysis targets indirect-readout geometry/flexibility (AVG equilibrium and
  DIAG stiffness values from MD tetramer tables) rather than chemical-group electrostatics
- Sequences contain non-standard or modified nucleobases outside the four canonical
  bases covered by the integer acceptor/donor/hydrophobic coding scheme
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: DNAffinity
  role: Provides the integer electrostatic base-coding scheme (acceptor, donor, hydrophobic
    groups) and the feature-assembly code whose output feeds {method}_regressor.py
  repo: https://github.com/Jalbiti/DNAffinity
- name: BigNASim
  role: Source of the MD-derived tetramer tables from which the indirect-readout block
    accompanying the direct-readout block is extracted (AVG and DIAG entries for six
    helical coordinates)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:99fa4946370d7179ac35caf4d070aff69b524424888337128747136798f43f7c
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
  - BigNASim
  - DNAffinity
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# dna-electrostatic-direct-readout-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes the direct-readout (chemical-group) component of DNAffinity features by assigning, per base-pair step, an integer electrostatic scheme coding acceptor, donor, and hydrophobic chemical groups, then column-summing base codes over each tetramer to yield five values per tetramer.

## When to use

Use when building DNAffinity feature matrices from DNA sequences and the direct-readout electrostatic block must be derived per tetramer, or when validating the encoding against the AACT worked example.

## When NOT to use

- Input is already a DNAffinity feature matrix or a precomputed tetramer feature table — the encoding step is already done
- The analysis targets indirect-readout geometry/flexibility (AVG equilibrium and DIAG stiffness values from MD tetramer tables) rather than chemical-group electrostatics
- Sequences contain non-standard or modified nucleobases outside the four canonical bases covered by the integer acceptor/donor/hydrophobic coding scheme

## Inputs

- DNA sequences from proteins/{protein}/{protein}_training.txt (uPBM), proteins/{protein}/{cycle}.txt (HT-SELEX), or proteins/{protein}/{protein}_{concentration}.txt plus proteins/{protein}/_freq_matrix_6.txt (gcPBM)
- Integer electrostatic base-coding scheme (acceptor, donor, hydrophobic chemical groups) from the DNAffinity package

## Outputs

- Five column-summed electrostatic (direct-readout) values per tetramer per sequence
- Direct-readout feature block assembled with tetramer-count and indirect-readout blocks into the per-sequence feature matrix used by {method}_regressor.py

## How to apply

After loading the sequences from the method-specific input files (e.g. proteins/{protein}/{protein}_training.txt for uPBM, {cycle}.txt for HT-SELEX, or {protein}_{concentration}.txt with _freq_matrix_6.txt for gcPBM, formatted per the shipped templates), slide a 4-mer window across each sequence to obtain overlapping tetramer occurrences. For each tetramer, assign the direct-readout electrostatic feature per base step using the integer scheme coding acceptor, donor, and hydrophobic chemical groups. Column-sum the four base codes so each tetramer contributes five electrostatic values; the paper's worked example sums the codes of 'AACT' to give [-2, 2, -2, -2, 1]. Assemble this block alongside the tetramer-count composition block and the indirect-readout block (AVG equilibrium values and DIAG stiffness entries for shift, slide, rise, tilt, roll, twist from the MD-derived BigNASim tetramer tables) into the per-sequence feature matrix consumed by {method}_regressor.py.

## Related tools

- **DNAffinity** (Provides the integer electrostatic base-coding scheme (acceptor, donor, hydrophobic groups) and the feature-assembly code whose output feeds {method}_regressor.py) — https://github.com/Jalbiti/DNAffinity
- **BigNASim** (Source of the MD-derived tetramer tables from which the indirect-readout block accompanying the direct-readout block is extracted (AVG and DIAG entries for six helical coordinates))

## Examples

```
python upbm_regressor.py Gata4
```

## Evaluation signals

- Reproduce the paper's worked example: summing the four base codes of 'AACT' must give the electrostatic feature vector [-2, 2, -2, -2, 1]
- Each tetramer yields exactly five electrostatic values (the column sums over its four base codes)
- Direct-readout block dimensions match the number of tetramers counted by the sliding 4-mer window across each sequence
- Input files match the formatting of the shipped DNAffinity templates (e.g. proteins/{protein}/{protein}_training.txt for upbm_regressor.py) before feature assembly
- The assembled feature matrix is accepted by {method}_regressor.py when the protein name is passed as the 1st argument (and cycle/concentration as the 2nd where applicable)

## Limitations

- The integer coding scheme covers the chemical groups of the four canonical nucleobases; extension to epigenetic variants, mismatches, mutations, or non-coding nucleobases is proposed but is not what the shipped encoding computes
- The concept of a transcription factor binding site is not uniquely defined and depends on the noisy, low-resolution experimental technique used to detect it, so the encoded features cannot support a universal predictor
- Extrapolating in vitro-trained predictions (built on these features) to in vivo settings is complicated by factors beyond intrinsic transcription factor affinity

## Evidence

- [other] Direct-readout electrostatics are encoded as five column-summed values per tetramer; the paper's worked example for 'AACT' sums four base codes to give the vector [–2, 2, –2, –2, 1].: [evidence span withheld — anchor status: not_found]
- [other] Assign the direct-readout electrostatic feature per base step using the integer scheme coding acceptor, donor, and hydrophobic chemical groups.: [evidence span withheld — anchor status: not_found]
