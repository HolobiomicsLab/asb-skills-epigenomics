---
name: tad-similarity-benchmark-evaluation
description: Use when called TAD boundaries (e.g., from TOAST, Armatus, Arrowhead,
  TopDom, HiCSeg, TADtree, PSYCHIC, or TADbit) must be compared against ground-truth
  TADs of simulated Hi-C genomes to rank methods or network configurations by FMS
  and Rand Index.
when_to_use_negative:
- When no ground-truth TADs exist for the input genome (e.g., real GM12878 Hi-C data)
  — use ChIP-seq/CTCF anchor enrichment instead of FMS/RI similarity
- When the TAD calls under evaluation include contiguous same-label regions shorter
  than 200 Kb, which the TOAST definition excludes as TADs
- When the goal is raw TAD identification from contact matrices rather than benchmarking
  of already-called TAD sets
tools:
- name: TOAST
  role: Source of the evaluation scripts (evaulate_TAD-simnarity.py) that compute
    FMS and Rand Index similarity between called and ground-truth TADs
  repo: https://github.com/ghaiyan/TOAST
- name: HDBSCAN
  role: Upstream clustering that assigns labels to genomic bins so contiguous same-label
    regions can be called as TADs before similarity scoring
  repo: https://github.com/ghaiyan/TOAST
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:6026bfaeace8196c0453398421710ddbc6223494bd3bdf0deab953e577526318
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ghaiyan/TOAST
  repo_url: https://github.com/ghaiyan/TOAST
  tools:
  - HDBSCAN
  - Juicer
  - NumPy
  - TOAST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# tad-similarity-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark-evaluation skill that quantifies similarity between algorithmically called TADs and ground-truth TADs using the Fowlkes-Mallows score (FMS) and Rand Index (RI), as done in the TOAST paper to validate TAD calls on simulated Trussart et al. and Wang et al. Hi-C datasets.

## When to use

Use when called TAD boundaries (e.g., from TOAST, Armatus, Arrowhead, TopDom, HiCSeg, TADtree, PSYCHIC, or TADbit) must be compared against ground-truth TADs of simulated Hi-C genomes to rank methods or network configurations by FMS and Rand Index.

## When NOT to use

- When no ground-truth TADs exist for the input genome (e.g., real GM12878 Hi-C data) — use ChIP-seq/CTCF anchor enrichment instead of FMS/RI similarity
- When the TAD calls under evaluation include contiguous same-label regions shorter than 200 Kb, which the TOAST definition excludes as TADs
- When the goal is raw TAD identification from contact matrices rather than benchmarking of already-called TAD sets

## Inputs

- Called TAD boundary file (TOAST merge-file output) from a candidate method or network configuration
- Ground-truth TAD annotations of a simulated genome (Trussart et al. toy genomes or Wang et al. simulated Hi-C dataset)
- NxN tab-separated intra-chromosomal contact matrices derived with juicer_tools from .hic files

## Outputs

- Fowlkes-Mallows score (FMS) per compared TAD set
- Rand Index (RI) per compared TAD set
- Ranked comparison table (e.g., gae_architecture_comparison.csv) of FMS/RI across methods or GAE variants

## How to apply

Convert called and ground-truth TAD sets into comparable boundary/region representations, then run the TOAST evaluation script evaulate_TAD-simnarity.py, which compares similarity between two TAD sets and reports the Fowlkes-Mallows score (FMS) and Rand Index (RI). Apply the shared TAD definition first: contiguous same-label regions longer than 200 Kb are identified as TADs, so filter short spurious regions before scoring. On the Wang et al. simulated data with five noise levels, expect FMS to decrease with noise but remain around 0.98, with TAD-number differences of 1-8 versus ground truth and average-length differences up to 5 Kb. Use the scores to rank configurations, e.g., in the GAE architecture sweep where the 2-layer network with hidden_dim=64 and output_dim=16 was optimal and most robust (a single-layer network with output_dim=128 was also competitive). Write the ranked comparison to a table (e.g., gae_architecture_comparison.csv) so the selected optimum can be traced to the evaluation evidence.

## Related tools

- **TOAST** (Source of the evaluation scripts (evaulate_TAD-simnarity.py) that compute FMS and Rand Index similarity between called and ground-truth TADs) — https://github.com/ghaiyan/TOAST
- **HDBSCAN** (Upstream clustering that assigns labels to genomic bins so contiguous same-label regions can be called as TADs before similarity scoring) — https://github.com/ghaiyan/TOAST

## Examples

```
python evaulate_TAD-simnarity.py  # compare the similarity between called TADs and ground-truth simulated-genome TADs (FMS, Rand Index)
```

## Evaluation signals

- FMS values on Wang et al. simulated data should remain around 0.98 across noise levels, decreasing slightly with increasing noise
- Difference in TAD number versus ground truth should stay in the 1-8 range and average TAD length difference within about 5 Kb
- Ranked comparison should show the 2-layer GAE with hidden_dim=64 and output_dim=16 (or 1-layer with output_dim=128) achieving the highest FMS/RI
- All scored TADs must come from contiguous same-label regions longer than 200 Kb
- evaulate_TAD-simnarity.py runs without error on every TAD set in the sweep and writes the comparison table

## Limitations

- FMS and RI require ground-truth TADs, so this evaluation is only possible on simulated genomes (e.g., Trussart et al. six artificially generated genomes, Forcato et al., Wang et al. five noise levels); real GM12878 data cannot be scored this way
- Similarity scores degrade as simulated noise level increases, so conclusions are noise-level dependent
- The TAD definition (contiguous same-label regions exceeding 200 Kb) means very short TADs are excluded from scoring, which can inflate similarity for methods that only call long TADs

## Evidence

- [readme] evaulate_TAD-simnarity.py:compare the similarity between TADs.: [evidence span withheld — anchor status: not_found]
