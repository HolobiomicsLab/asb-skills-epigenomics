---
name: regulatory-element-coordinate-processing
description: Use when you have a set of genomic intervals (REM BED-like records with
  chr, start, end) and need to group overlapping/adjacent elements into clusters and
  report cluster-level statistics.
when_to_use_negative:
- The goal is motif or TF enrichment on REM sequences (e.g. PASTAA on bedtools getfasta
  output) rather than interval clustering
- The input intervals are TF ChIP-seq peaks being queried against REMs via the Region
  Query with 50% overlap — use EpiRegio's overlap query instead of CREM construction
- Only single, non-overlapping REM intervals are of interest and no clustering or
  span statistics are required
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: EpiRegio
  role: 'Source of the REM dataset and CREM definition: REMs are grouped into CREMs
    with unique IDs based on overlap/adjacency, starting from a gene rather than the
    reverse'
  repo: https://epiregio.de/
- name: STITCHIT
  role: Algorithm underlying EpiRegio that interprets chromatin accessibility data
    with respect to gene expression variation, producing the REMs that are then clustered
    into CREMs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:dbc801d98ca062cb470d44ad4c5641269d5f6efae699b7dcb500fdced25c83d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - STITCHIT
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# regulatory-element-coordinate-processing

## Summary

Builds Clusters of Regulatory EleMents (CREMs) by merging overlapping or adjacent REM intervals from the EpiRegio database per chromosome, and computes aggregate statistics (CREM count, average REMs per CREM, mean span, complete-overlap clusters) used to characterize the REM set.

## When to use

Use when you have a set of genomic intervals (REM BED-like records with chr, start, end) and need to group overlapping/adjacent elements into clusters and report cluster-level statistics.

## When NOT to use

- The goal is motif or TF enrichment on REM sequences (e.g. PASTAA on bedtools getfasta output) rather than interval clustering
- The input intervals are TF ChIP-seq peaks being queried against REMs via the Region Query with 50% overlap — use EpiRegio's overlap query instead of CREM construction
- Only single, non-overlapping REM intervals are of interest and no clustering or span statistics are required

## Inputs

- EpiRegio REM dataset from Zenodo record 3758494: 2,404,861 REM records with chr, start, end coordinates and REM IDs
- REM-gene assignments (the same interval can carry multiple REM IDs)

## Outputs

- CREM table with unique CREM IDs, member REM IDs, REM count per CREM, and CREM span in bp
- crem_statistics.csv: summary table with total CREMs, average REMs per CREM, mean span ± standard deviation, and count of complete-overlap CREMs

## How to apply

Load the EpiRegio REM dataset (all 2,404,861 REMs with chr, start, end and REM IDs) from the deposited Zenodo record 3758494, retaining REM-gene assignments since the same interval can carry multiple REM IDs. Sort REMs by chromosome and start coordinate, then merge on each chromosome using the CREM rule: intervals overlapping by at least 1 bp or directly adjacent with a 0 bp gap belong to one cluster, and only clusters composed of a minimum of two REMs are retained as CREMs with unique IDs. For each CREM compute the number of member REMs and the CREM span (max end minus min start) in bp, and separately count CREMs with complete overlap, i.e. clusters in which every REM fully overlaps all other REMs in the cluster. Aggregate the summary statistics and compare against the reported values (365,286 CREMs; 3.5 REMs per CREM; 534 ± 460 bp average span; 9,786 complete-overlap CREMs) to validate correctness.

## Related tools

- **EpiRegio** (Source of the REM dataset and CREM definition: REMs are grouped into CREMs with unique IDs based on overlap/adjacency, starting from a gene rather than the reverse) — https://epiregio.de/
- **STITCHIT** (Algorithm underlying EpiRegio that interprets chromatin accessibility data with respect to gene expression variation, producing the REMs that are then clustered into CREMs)

## Examples

```
sort -k1,1 -k2,2n rem.bed | bedtools merge -d 0 -c 4 -o count > crem.bed && python scripts/crem_statistics.py --input crem.bed --output crem_statistics.csv
```

## Evaluation signals

- Total CREM count must equal 365,286 when applying the CREM rule to the full REM set from Zenodo record 3758494
- Average number of REMs per CREM must equal 3.5
- Mean CREM span must equal 534 bp with standard deviation 460 bp
- Count of CREMs in which all contained REMs overlap completely must equal 9,786
- No retained cluster may contain fewer than two REMs, and every merged pair must overlap by ≥1 bp or be exactly adjacent (0 bp gap)

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification of the actual regulation of genes by their REMs
- Only clusters composed of a minimum of two REMs are retained as CREMs — singleton clusters are discarded by definition
- The same genomic interval can carry multiple REM IDs (multiple REM-gene assignments), which must be preserved during sorting/merging or REM counts per CREM will be wrong

## Evidence

- [results] Applying the CREM definition to the REM set yields 365,286 distinct CREMs containing 3.5 REMs on average spanning 534 bp (±460 bp): [evidence span withheld — anchor status: not_found]
- [methods] Merge REMs on each chromosome into clusters using the CREM rule: intervals overlapping by at least 1 bp or directly adjacent with a 0 bp gap belong to one cluster: [evidence span withheld — anchor status: not_found]
- [methods] only clusters composed of a minimum of two REMs are retained as CREMs with unique IDs: [evidence span withheld — anchor status: not_found]
- [intro] it starts with a gene and looks for REMs, not viceversa: "In contrast to existing approaches, it starts with a gene and looks for REMs, not viceversa."
- [readme] EpiRegio is a resource of Regulatory EleMents (REMs) associated to protein coding, and non-protein coding genes: "EpiRegio is a resource of Regulatory EleMents (REMs) associated to protein coding, and non-protein coding genes."
