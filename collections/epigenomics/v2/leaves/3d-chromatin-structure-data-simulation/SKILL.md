---
name: 3d-chromatin-structure-data-simulation
description: Use when benchmarking single-cell Hi-C TAD/TLD predictors against a known
  reference partition, or when ground-truth-free experimental scHi-C data cannot disambiguate
  predictor accuracy.
when_to_use_negative:
- The goal is to call TLDs on real experimental scHi-C data — simulation is only for
  controlled benchmarking, not for biological discovery
- The input is already experimental scHi-C with an accepted gold-standard annotation,
  making synthetic ground truth unnecessary
- Bulk Hi-C data is the target, since simulation was designed to mimic sparse single-cell
  contact matrices rather than bulk depths
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_0622
tools:
- name: deDoc2
  role: Downstream TLD predictor benchmarked on the simulated single-cell Hi-C matrices;
    achieved highest AMI and WS at D = 750 and D = 1000
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Comparison predictor (bulk-oriented structural-entropy tool) evaluated on
    the simulated cells
  repo: https://github.com/yinxc/structural-information-minimisation
- name: Higashi
  role: Embedding-based comparison predictor run on simulated data with nbr = 0, embedding_epoch
    = 1, no_nbr_epoch = 20; TADs called via its call_tads function
- name: deTOKI
  role: NMF-based comparison predictor run with default settings on the simulated
    matrices
- name: scHiCluster
  role: Imputation/clustering-based comparison predictor run with default settings
    on the simulated matrices
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:04cca2f808e4850e7457d5c0fdeee459512727ae120feee3dc60f3ec7700f0f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - Higashi
  - deDoc
  - deDoc2
  - deTOKI
  - scHiCluster
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# 3d-chromatin-structure-data-simulation

## Summary

Simulate single-cell Hi-C contact matrices from a 3D chromosome structure ensemble (IMP-based) with distance-dependent sampling weight Weight(i, j) = D − distance(i, j), producing ground-truth-controlled benchmark data at 40 kb resolution for evaluating TLD predictors such as deDoc2.

## When to use

Use when benchmarking single-cell Hi-C TAD/TLD predictors against a known reference partition, or when ground-truth-free experimental scHi-C data cannot disambiguate predictor accuracy.

## When NOT to use

- The goal is to call TLDs on real experimental scHi-C data — simulation is only for controlled benchmarking, not for biological discovery
- The input is already experimental scHi-C with an accepted gold-standard annotation, making synthetic ground truth unnecessary
- Bulk Hi-C data is the target, since simulation was designed to mimic sparse single-cell contact matrices rather than bulk depths

## Inputs

- 3D chromosome structure ensemble (≈100 single-cell structures, e.g., IMP-derived models) for a target region such as chr18:50–55 Mb
- Simulation threshold parameter D (e.g., 750 or 1000) defining sampling weight Weight(i, j) = D − distance(i, j)
- Resolution setting (40 kb binsize) for binning simulated contacts

## Outputs

- ~100 simulated single-cell intra-chromosomal Hi-C contact matrices (n × n at 40 kb binsize) per threshold D
- Reference TLD partitions (K) per simulated cell derived from the structural ensemble
- Per-predictor, per-threshold accuracy table of AMI (and WS) scores

## How to apply

Generate a 3D structure ensemble (~100 single cells) for a defined genomic region — in the deDoc2 paper, a 5 Mb region of chr18:50–55 Mb was randomly chosen. For each simulated cell, sample contacts between loci i and j with probability weight Weight(i, j) = D − distance(i, j), where D is the simulation threshold controlling contact decay with 3D distance (the paper used D = 750 and D = 1000). Bin the sampled contacts into intra-chromosomal contact matrices at 40 kb resolution to mimic sparse single-cell data. Because the ensemble defines the reference 3D organization, the simulated cells come with reference partitions (K) against which predicted TLD partitions (T) can be scored with adjusted mutual information (sklearn.metrics.adjusted_mutual_info_score) and a weighted score (WS), including all predicted TLDs and intermediate windows. Vary D to probe robustness of predictors across different contact-decay regimes.

## Related tools

- **deDoc2** (Downstream TLD predictor benchmarked on the simulated single-cell Hi-C matrices; achieved highest AMI and WS at D = 750 and D = 1000) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Comparison predictor (bulk-oriented structural-entropy tool) evaluated on the simulated cells) — https://github.com/yinxc/structural-information-minimisation
- **Higashi** (Embedding-based comparison predictor run on simulated data with nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20; TADs called via its call_tads function)
- **deTOKI** (NMF-based comparison predictor run with default settings on the simulated matrices)
- **scHiCluster** (Imputation/clustering-based comparison predictor run with default settings on the simulated matrices)

## Examples

```
java -jar deDoc2.jar -inputfile ./sim_chr18_50_55Mb_D750_cell001.matrix -binsize 40; from sklearn.metrics import adjusted_mutual_info_score; ami = adjusted_mutual_info_score(T, K)
```

## Evaluation signals

- The reproduced AMI ranking on the simulated datasets must place deDoc2 first at both D = 750 and D = 1000, matching the reported benchmark
- Each simulated cell yields a complete 40 kb-binned contact matrix for the chr18:50–55 Mb region with the expected number of bins for a 5 Mb region
- AMI is computed with sklearn.metrics.adjusted_mutual_info_score over partitions T and K that include all predicted TLDs and intermediate windows, so scores are comparable across predictors
- Changing D from 750 to 1000 changes the contact-decay regime; results should be reported per threshold rather than pooled
- Higashi-specific settings (nbr = 0, embedding_epoch = 1, no_nbr_epoch = 20) are applied only to simulated data; defaults elsewhere would invalidate the comparison

## Limitations

- Simulation depends on the chosen 3D structure ensemble; conclusions about predictor accuracy are conditional on the IMP-based ensemble and the chr18:50–55 Mb region sampled
- Only two simulation thresholds (D = 750, D = 1000) were exercised in the benchmark; other decay regimes were not validated
- Embedding-based predictors need k-means on reduced embeddings to assign partitions, adding a clustering step that can affect comparability
- The article does not provide data accession IDs, software versions, or the simulation code, so exact reproduction requires reconstructing the sampling procedure from the stated weight formula

## Evidence

- [results] Simulated ~100 single-cell Hi-C data from a 3D chromosome structure ensemble: "We simulated Hi‐C data from a 3D chromosome structure ensemble containing ≈100 single cells"
- [results] Region choice for simulation: "a 5 Mb‐long genome region, i.e., chr18:50–55 Mb, was randomly chosen as an example"
- [results] Finding: deDoc2 most accurate on simulated data: "deDoc2 exhibits first‐class performance on the metrics we assessed in the downsampled, simulated, and experimental scHi‐C data"
- [methods] Simulation weight function and resolution from the workflow: "All test TLD predictors were run with default settings, except for Higashi"
- [discussion] deDoc2 runs without imputation, relevant for sparse simulated inputs: "deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation, while most state‐of‐the‐art TLD predictors do"
- [readme] README input format for the simulated matrices: "deDoc2 supports both contact matrix format and sparse format as in deDoc. deDoc2 needs no normalization process, raw Hi-C data is suitable."
