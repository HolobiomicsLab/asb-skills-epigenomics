---
name: base-pair-step-stiffness-parameter-mapping
description: Use when preprocessing DNA sequences (uPBM, HT-SELEX, or gcPBM) into
  per-sequence feature matrices for DNAffinity affinity prediction, requiring tetramer
  counts and base-pair-step structural/electrostatic encoding.
when_to_use_negative:
- The sequences are already encoded as a feature matrix and only the regressor needs
  to be run (e.g. an existing Gata4_training.txt-processed pipeline).
- The analysis targets regression performance or competitor comparison (DeepBind,
  DNAShapeR, BEESEM, BindSpace) rather than feature construction.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Package providing the tetramer sliding-window counting, BigNASim-based indirect-readout
    lookup tables, and integer direct-readout electrostatic coding; runs via {method}_regressor.py
  repo: https://github.com/Jalbiti/DNAffinity
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:6f96faefeded39e6df5136d7056019355d67cee1fda81de7bb7a848eec182db5
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# base-pair-step-stiffness-parameter-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes DNA sequences for DNAffinity by counting overlapping tetramers and mapping each base-pair step to indirect-readout structural parameters (AVG equilibrium geometry and DIAG stiffness from BigNASim MD tables) plus direct-readout electrostatic features. This converts raw sequences into the physical-property feature matrix consumed by the random forest regressor.

## When to use

Use when preprocessing DNA sequences (uPBM, HT-SELEX, or gcPBM) into per-sequence feature matrices for DNAffinity affinity prediction, requiring tetramer counts and base-pair-step structural/electrostatic encoding.

## When NOT to use

- The sequences are already encoded as a feature matrix and only the regressor needs to be run (e.g. an existing Gata4_training.txt-processed pipeline).
- The analysis targets regression performance or competitor comparison (DeepBind, DNAShapeR, BEESEM, BindSpace) rather than feature construction.

## Inputs

- proteins/{protein}/{protein}_training.txt (uPBM training sequences)
- proteins/{protein}/{cycle}.txt (HT-SELEX cycle sequences)
- proteins/{protein}/{protein}_{concentration}.txt (gcPBM sequences)
- proteins/{protein}/_freq_matrix_6.txt (gcPBM frequency matrix)
- MD-derived tetramer tables of the BigNASim database (AVG equilibrium and DIAG stiffness values)

## Outputs

- Per-sequence tetramer-count composition features
- Per-tetramer indirect-readout feature block (six helical coordinates: shift, slide, rise, tilt, roll, twist with AVG and DIAG entries)
- Per-tetramer direct-readout electrostatic feature vector (five column-summed values, e.g. [-2, 2, -2, -2, 1] for AACT)
- Assembled feature matrix passed to {method}_regressor.py

## How to apply

First compute the tetramer-count composition feature by sliding a 4-mer window over each sequence and counting overlapping tetranucleotide occurrences. For each tetramer, look up the indirect-readout base-pair-step parameters in the MD-derived tetramer tables of the BigNASim database, extracting the AVG equilibrium values and DIAG stiffness diagonal entries for the six helical coordinates (shift, slide, rise, tilt, roll, twist). Assign the direct-readout electrostatic feature per base step using the integer scheme coding acceptor, donor, and hydrophobic chemical groups; direct-readout electrostatics are column-summed to five values per tetramer. Assemble the three feature blocks (tetramer composition, indirect readout, direct readout) into the per-sequence feature matrix consumed by {method}_regressor.py, invoked with the protein name as the 1st argument and the cycle number (HT-SELEX) or concentration (gcPBM) as the 2nd argument where applicable.

## Related tools

- **DNAffinity** (Package providing the tetramer sliding-window counting, BigNASim-based indirect-readout lookup tables, and integer direct-readout electrostatic coding; runs via {method}_regressor.py) — https://github.com/Jalbiti/DNAffinity

## Examples

```
python selex_regressor.py Gata4 4
```

## Evaluation signals

- Reproduce the worked AACT example: its direct-readout electrostatic feature vector must equal [-2, 2, -2, -2, 1].
- Tetramer counts cover all 256 tetranucleotides; sliding-window counts include overlapping occurrences, so count totals per sequence equal sequence length minus 3.
- Indirect-readout table lookups return six helical coordinates (shift, slide, rise, tilt, roll, twist) with both AVG equilibrium and DIAG stiffness diagonal entries per tetramer.
- Input sequence files match the shipped templates (upbm_training.txt, {cycle}.txt, {protein}_{concentration}.txt plus _freq_matrix_6.txt) before feature computation.
- Downstream regressor accepts the assembled feature matrix and reports R2 consistent with paper baselines (gcPBM ~0.93 ± 0.02, uPBM ~0.69 ± 0.17).

## Limitations

- Indirect-readout parameters depend on the MD-derived BigNASim tetramer tables; accuracy is bounded by the simulation-derived parameter database rather than protein-specific data.
- The transcription factor binding site concept is not uniquely defined, as it depends on the intrinsically noisy and low-resolution experimental technique used to detect it.
- HT-SELEX inputs may require SVM filtering of cases displaying inconsistency between the enrichments at different cycles before feature computation.

## Evidence

- [other] Direct-readout electrostatics are encoded as five column-summed values per tetramer; the paper's worked example for 'AACT' sums four base codes to give the vector [–2, 2, –2, –2, 1].: [evidence span withheld — anchor status: not_found]
- [other] For each tetramer, look up the indirect-readout base-pair-step parameters in the MD-derived tetramer tables of the BigNASim database.: [evidence span withheld — anchor status: not_found]
- [intro] These properties consider both the equilibrium geometry and the flexibility, as defined by Olson (42), to describe DNA conformation with sequence-dependence at a base-pair step (bps) resolution: "These properties consider both the equilibrium geometry and the flexibility, as defined by Olson (42), to describe DNA conformation with sequence-dependence at a base-pair step (bps) resolution"
- [readme] 1. Make sure you have all the required files (compare formatting with the templates): "Make sure you have all the required files (compare formatting with the templates)"
