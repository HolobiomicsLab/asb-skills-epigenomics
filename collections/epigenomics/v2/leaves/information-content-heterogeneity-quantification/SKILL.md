---
name: information-content-heterogeneity-quantification
description: Use when you have CscoreTool-M *_cscore.txt probability outputs and need
  to measure how heterogeneous sub-compartment localization is across cells, e.g.
  comparing proliferating vs terminally differentiated cells or G1 vs S/G2 stages.
when_to_use_negative:
- Input is bulk Hi-C from a single homogeneous cell population with no cell-cycle
  or cell-type mixture to compare — heterogeneity quantification over probability
  scores has little meaning
- You need discrete sub-compartment labels or GO enrichment of sub-compartment genes
  rather than a continuous heterogeneity measure per window
- The *_cscore.txt probabilities were computed with mismatched N_subcompartment across
  conditions being compared (e.g. 5 vs 10), since entropy values are not comparable
  across different numbers of sub-compartments
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3912
tools:
- name: CscoreTool-M
  role: Infers per-window sub-compartment probability scores (cscore.txt) from Hi-C
    summary files, which are the direct input to Shannon information content computation
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Compiles CscoreTool-M.cpp and twister.cpp with -fopenmp -O3 if the prebuilt
    single-file executable does not work
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:c9ad447dc9170a56ca0263910e58d17a668b733d4728b03d9b5b911aa92eee7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - g++
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3912
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# information-content-heterogeneity-quantification

## Summary

Quantifies genome-organization heterogeneity within a cell population by computing Shannon information content over CscoreTool-M sub-compartment probability scores (from *_cscore.txt outputs) per genomic window, then aggregating by cell type and cell-cycle stage.

## When to use

Use when you have CscoreTool-M *_cscore.txt probability outputs and need to measure how heterogeneous sub-compartment localization is across cells, e.g. comparing proliferating vs terminally differentiated cells or G1 vs S/G2 stages.

## When NOT to use

- Input is bulk Hi-C from a single homogeneous cell population with no cell-cycle or cell-type mixture to compare — heterogeneity quantification over probability scores has little meaning
- You need discrete sub-compartment labels or GO enrichment of sub-compartment genes rather than a continuous heterogeneity measure per window
- The *_cscore.txt probabilities were computed with mismatched N_subcompartment across conditions being compared (e.g. 5 vs 10), since entropy values are not comparable across different numbers of sub-compartments

## Inputs

- CscoreTool-M *_cscore.txt outputs: per-genomic-window sub-compartment probability scores
- CscoreTool-M *_cscore.bedgraph outputs with c-scores normalized to sum to 1 per genomic region (optional alternative input)
- Hi-C summary files in HOMER runHiCpca.pl HiCsummary format for each cell type or cell-cycle stage
- Equal-length windows_Rabl.bed file with Rabl position (relative centromere-telomere position) in column 4

## Outputs

- Per-genomic-window Shannon information content values over the 5- or 10-sub-compartment probability vectors
- Aggregated heterogeneity metrics per cell type (e.g. OSN vs proliferating) and cell-cycle stage (G1 vs S/G2) supporting the heterogeneity ordering comparison

## How to apply

Run CscoreTool-M with N_subcompartment set to the desired resolution (5 for cell-cycle-sorted Nagano 2017 scHi-C, 10 for HCT116 Rao 2017 data), N_Rablwindow=20, and N_session=12, passing hg38_blacklisted.bed and Test_excluded.txt to exclude ENCODE blacklist regions and translocation-mediated interactions. From each *_cscore.txt output, which reports the Cscore estimated per genomic window as the probability of the region locating in a specific sub-compartment, compute the Shannon information content (entropy) over the probability scores for each genomic window; alternatively use the normalized *_cscore.bedgraph file where c-scores add up to 1 per region. Aggregate the per-window information content by condition (e.g. OSN vs proliferating cells, G1 vs S/G2) to reproduce the reported heterogeneity ordering: proliferating cells show higher genome organization heterogeneity than terminally differentiated non-proliferating cells, and G1 shows lower heterogeneity than S/G2, likely because cells at different cell-cycle stages mix within the population.

## Related tools

- **CscoreTool-M** (Infers per-window sub-compartment probability scores (cscore.txt) from Hi-C summary files, which are the direct input to Shannon information content computation) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Compiles CscoreTool-M.cpp and twister.cpp with -fopenmp -O3 if the prebuilt single-file executable does not work)

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_3 5 20 12 hg38_blacklisted.bed Test_excluded.txt
```

## Evaluation signals

- Reproduced heterogeneity ordering: proliferating cells show higher Shannon information content than terminally differentiated non-proliferating cells (OSN), and G1 shows lower information content than S/G2
- Per-region normalization check: c-scores in *_cscore.bedgraph add up to 1 for each genomic region; regions with no reads are absent
- Number of probability columns in *_cscore.txt matches the N_subcompartment parameter used (5 or 10)
- Heterogeneity differences hold after applying hg38_blacklisted.bed and Test_excluded.txt filtering (7th and 8th program arguments)

## Limitations

- Heterogeneity signal is confounded by cell-cycle mixture — the paper attributes higher heterogeneity in proliferating cells to cells at different cell-cycle stages, so G1 vs S/G2 differences reflect stage composition, not only genuine structural variability
- Regions with no Hi-C reads are not shown in the normalized bedgraph and contribute no information content, so coverage gaps bias genome-wide aggregates
- N_session (12 in the paper) depends on available computational resources and can affect convergence of the probability estimates

## Evidence

- [abstract] The compartment scores calculated by CscoreTool-M also help to quantify the levels of heterogeneity in sub-compartment localization within cell populations.: "The compartment scores calculated by CscoreTool-M also help to quantify the levels of heterogeneity in sub-compartment localization within cell populations."
- [abstract] proliferating vs non-proliferating heterogeneity: "By comparing proliferating cells and terminally differentiated non-proliferating cells, we show that the proliferating cells have higher genome organization heterogeneity, which is likely caused by"
- [readme] output file description: "XXX_cscore.txt This is the Cscore estimated for each genomic window."
- [readme] normalized bedgraph output: "XXX_cscore.bedgraph This is the bedgraph file made for visualization. Ths c-scores are normalized to add up to 1 for each genomic region. Regions with no reads are not shown."
- [methods] example run with filtering: "./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_3 5 20 12 hg38_blacklisted.bed Test_excluded.txt"
