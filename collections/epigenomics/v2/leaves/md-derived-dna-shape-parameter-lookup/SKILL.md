---
name: md-derived-dna-shape-parameter-lookup
description: Use when converting raw DNA sequence files (uPBM, HT-SELEX, or gcPBM)
  into physics-based features — tetramer counts, BigNASim MD-derived bps geometry/stiffness,
  and base-step electrostatic codes — for binding-affinity prediction.
when_to_use_negative:
- The input is already a per-sequence feature matrix or shape-parameter table, so
  tetramer decomposition and BigNASim lookup would duplicate existing features
- The goal is motif discovery or sequence trimming (e.g. MEME-suite based trimming
  of uPBM sequences) rather than physics-based feature construction for the random-forest
  regressor
- The sequences contain bases or modifications outside the ACGT tetramer tables, which
  the paper's direct/indirect readout scheme does not cover
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: DNAffinity
  role: Package containing the feature-extraction code, tetramer parameter database,
    and the {method}_regressor.py entry points
  repo: https://github.com/Jalbiti/DNAffinity
- name: BigNASim
  role: 'Source of the MD-derived tetramer tables: AVG equilibrium values and DIAG
    stiffness diagonal entries for the six base-pair-step helical coordinates'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:407eff4fd3248e4ab1d491437c4f111d91a5f39da2a6a21165596849f79c239e
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# MD-derived DNA shape parameter lookup

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Feature-extraction step of DNAffinity: each DNA sequence is decomposed into overlapping tetramers and annotated with indirect-readout structural parameters (equilibrium geometry and flexibility from MD simulations) plus direct-readout electrostatic codes, producing the per-sequence feature matrix consumed by the random-forest regressors.

## When to use

Use when converting raw DNA sequence files (uPBM, HT-SELEX, or gcPBM) into physics-based features — tetramer counts, BigNASim MD-derived bps geometry/stiffness, and base-step electrostatic codes — for binding-affinity prediction.

## When NOT to use

- The input is already a per-sequence feature matrix or shape-parameter table, so tetramer decomposition and BigNASim lookup would duplicate existing features
- The goal is motif discovery or sequence trimming (e.g. MEME-suite based trimming of uPBM sequences) rather than physics-based feature construction for the random-forest regressor
- The sequences contain bases or modifications outside the ACGT tetramer tables, which the paper's direct/indirect readout scheme does not cover

## Inputs

- proteins/{protein}/{protein}_training.txt (uPBM sequence file)
- proteins/{protein}/{cycle}.txt (HT-SELEX sequence file)
- proteins/{protein}/{protein}_{concentration}.txt plus proteins/{protein}/_freq_matrix_6.txt (gcPBM sequence and frequency-matrix files)
- MD-derived tetramer tables of the BigNASim database (AVG equilibrium and DIAG stiffness diagonal entries)
- Direct-readout integer scheme coding acceptor, donor, and hydrophobic chemical groups per base step

## Outputs

- Per-sequence tetramer-count composition feature vector (overlapping 4-mer occurrences)
- Per-tetramer indirect-readout matrix of six helical coordinates (shift, slide, rise, tilt, roll, twist) with AVG equilibrium and DIAG stiffness diagonal values
- Per-tetramer direct-readout electrostatic vector (five column-summed values)
- Assembled per-sequence feature matrix consumed by {method}_regressor.py

## How to apply

Load the method-specific sequence files (proteins/{protein}/{protein}_training.txt for uPBM, {cycle}.txt for HT-SELEX, {protein}_{concentration}.txt plus _freq_matrix_6.txt for gcPBM) and check formatting against the shipped templates. Slide a 4-mer window across each sequence and count overlapping tetranucleotide occurrences to build the composition feature. For each tetramer, query the MD-derived tetramer tables of the BigNASim database, extracting the AVG equilibrium values and the DIAG stiffness diagonal entries for the six helical coordinates (shift, slide, rise, tilt, roll, twist). Assign the direct-readout electrostatic feature per base step using the integer scheme coding acceptor, donor, and hydrophobic chemical groups, column-summing five values per tetramer. Assemble the three blocks into the per-sequence feature matrix passed to {method}_regressor.py; validate against the worked AACT example, whose electrostatic vector must equal [-2, 2, -2, -2, 1].

## Related tools

- **DNAffinity** (Package containing the feature-extraction code, tetramer parameter database, and the {method}_regressor.py entry points) — https://github.com/Jalbiti/DNAffinity
- **BigNASim** (Source of the MD-derived tetramer tables: AVG equilibrium values and DIAG stiffness diagonal entries for the six base-pair-step helical coordinates)

## Examples

```
python upbm_regressor.py Gata4
```

## Evaluation signals

- The worked AACT example reproduces the direct-readout electrostatic vector [-2, 2, -2, -2, 1] by summing four base codes
- Every tetramer row carries values for exactly the six helical coordinates (shift, slide, rise, tilt, roll, twist) with both AVG equilibrium and DIAG stiffness diagonal entries
- Input file formatting matches the shipped templates before the regressor runs (proteins/{protein}/ file layout)
- Downstream sanity check: random-forest regressor trained on the assembled features should reproduce dataset-level performance (e.g. gcPBM average R2 of 0.93 ± 0.02, uPBM R2 = 0.69 ± 0.17)

## Limitations

- Feature parameters derive from first-principle molecular dynamics simulations; sequences or base-pair steps absent from the tetramer tables cannot be parameterized
- Downstream predictions inherit the paper's caveat that 'the concept of transcription factor binding site is not uniquely defined', since it depends on the noisy, low-resolution experimental technique (uPBM, gcPBM, HT-SELEX)
- In vitro-trained features may not transfer directly to in vivo settings where factors beyond intrinsic transcription factor affinity operate

## Evidence

- [methods] extracting the AVG equilibrium values and DIAG stiffness diagonal entries for the six helical coordinates (shift, slide, rise, tilt, roll, twist): [evidence span withheld — anchor status: not_found]
- [results] Direct-readout electrostatics are encoded as five column-summed values per tetramer; the paper's worked example for 'AACT' sums four base codes: [evidence span withheld — anchor status: not_found]
