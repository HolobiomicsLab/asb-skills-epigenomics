---
name: dna-tetramer-sliding-window-composition-counting
description: Use when raw DNA sequences from uPBM, HT-SELEX or gcPBM must be converted
  into fixed-length tetramer-count feature vectors to feed DNAffinity's random forest
  regressor.
when_to_use_negative:
- Input is already a feature matrix (e.g. _freq_matrix_6.txt supplied alongside gcPBM
  sequences) rather than raw sequences needing tetramer counting
- The goal is motif discovery or consensus finding rather than fixed-length physicochemical
  feature extraction for affinity regression
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Package providing the tetramer sliding-window counting code, formatting templates,
    and {method}_regressor.py entry points
  repo: https://github.com/Jalbiti/DNAffinity
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:80bbffbc4b4cf94087bf22df0af5200936ccc9fcef74dbeefa04b7f744ed79dd
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
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# dna-tetramer-sliding-window-composition-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-sequence tetranucleotide composition features for DNAffinity by sliding an overlapping 4-mer window across DNA sequences and counting tetramer occurrences, forming the sequence-composition feature block combined with indirect (MD-derived) and direct (electrostatic) readout features.

## When to use

Use when raw DNA sequences from uPBM, HT-SELEX or gcPBM must be converted into fixed-length tetramer-count feature vectors to feed DNAffinity's random forest regressor.

## When NOT to use

- Input is already a feature matrix (e.g. _freq_matrix_6.txt supplied alongside gcPBM sequences) rather than raw sequences needing tetramer counting
- The goal is motif discovery or consensus finding rather than fixed-length physicochemical feature extraction for affinity regression

## Inputs

- proteins/{protein}/{protein}_training.txt (uPBM sequence file)
- proteins/{protein}/{cycle}.txt (HT-SELEX cycle sequence file)
- proteins/{protein}/{protein}_{concentration}.txt (gcPBM sequence file)
- proteins/{protein}/_freq_matrix_6.txt (gcPBM frequency matrix)
- MD-derived tetramer tables of indirect-readout parameters (BigNASim)
- Direct-readout electrostatic base codes (acceptor/donor/hydrophobic groups)

## Outputs

- Per-sequence tetramer-count composition feature vectors
- Assembled per-sequence feature matrix combining composition, indirect-readout (6 helical coordinates AVG + DIAG stiffness), and direct-readout electrostatic blocks for the random forest regressor

## How to apply

Load the appropriate input file — proteins/{protein}/{protein}_training.txt for uPBM, proteins/{protein}/{cycle}.txt for HT-SELEX, or proteins/{protein}/{protein}_{concentration}.txt with proteins/{protein}/_freq_matrix_6.txt for gcPBM — checking formatting against the shipped templates. Slide a 4-mer window across each DNA sequence counting overlapping tetranucleotide occurrences to build the composition feature block. Combine this block with indirect-readout features from the BigNASim MD tetramer tables (AVG equilibrium values and DIAG stiffness diagonal entries for shift, slide, rise, tilt, roll, twist) and direct-readout electrostatic features per base step from the acceptor/donor/hydrophobic integer coding. Assemble the three feature blocks into the per-sequence matrix consumed by {method}_regressor.py, passing the protein name as the 1st argument and cycle number or concentration as the 2nd where applicable.

## Related tools

- **DNAffinity** (Package providing the tetramer sliding-window counting code, formatting templates, and {method}_regressor.py entry points) — https://github.com/Jalbiti/DNAffinity

## Examples

```
python upbm_regressor.py Gata4
```

## Evaluation signals

- Reproduce the worked AACT electrostatic example: summed four base codes must equal [-2, 2, -2, -2, 1]
- Feature matrix row length matches the expected tetramer count (256 tetramers) plus indirect/direct readout columns
- Input files pass format comparison against the templates shipped in the DNAffinity repository
- Downstream {method}_regressor.py runs without argument errors (protein name as 1st argument, cycle/concentration as 2nd)
- Resulting regressor performance falls within reported ranges (gcPBM average R2 0.93 ± 0.02; uPBM R2 0.69 ± 0.17)

## Limitations

- Sequences must match the expected file formats exactly; preprocessing (e.g. uPBM trimming based on MEME suite results) is done upstream in the notebooks folder
- Feature parameters depend on sequence variability explored by the experiment; the TFBS concept is not uniquely defined, so no universal predictor exists
- DNAffinity may predict many potential binding sites with no experimental in vivo binding evidence, especially when extrapolating in vitro to in vivo settings

## Evidence

- [other] Compute the tetramer-count composition feature by sliding a 4-mer window: [evidence span withheld — anchor status: not_found]
- [other] reproduce the worked AACT example: [evidence span withheld — anchor status: not_found]
