---
name: loop-anchor-overlap-verification
description: Use when SnapFISH loop summits have been called from chromatin tracing
  data and must be validated against known anchors (e.g., Sox2 promoter-super-enhancer)
  within a 10 Kb tolerance.
when_to_use_negative:
- Input is a count-based proximity-ligation assay dataset (e.g., bulk Hi-C); use HiCCUPS
  rather than SnapFISH output for anchor overlap checks
- Reference loop anchors are unknown or not genomically annotated; the 10 Kb overlap
  criterion requires pre-defined anchor coordinates
- Loop candidates have not yet been clustered into summits (Step 4a); singletons are
  removed and only summits should be checked against anchors
edam_operation: http://edamontology.org/operation_3903
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: SnapFISH
  role: Produces the loop summit and candidate tables whose anchors are checked for
    overlap with reference promoter-super-enhancer anchors within 10 Kb
  repo: https://github.com/HuMingLab/SnapFISH
- name: HiCCUPS
  role: Bulk Hi-C loop caller used as the orthogonal reference to confirm the SnapFISH
    summit is consistent with the known Sox2 enhancer-promoter loop
- name: pandas
  role: Loads and parses the output_loop_summit / output_loop_candidate text tables
    and the ANN.txt annotation for coordinate comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-40658-3
    title: 'SnapFISH: a computational pipeline to identify chromatin loops from multiplexed
      DNA FISH data'
  source_content_hash: sha256:ae4b1e712d29a26af75d41c8271cd6419091b2a5ba10b34ce8a42d4b3361290e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapFISH
  tools:
  - HiCCUPS
  - Python
  - SnapFISH
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3903
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# loop-anchor-overlap-verification

## Summary

Verification step that checks whether SnapFISH-called loop summits overlap predefined biological anchors, such as the Sox2 promoter–super-enhancer loop, within 10 Kb in both haplotypes. This grounds de novo loop calls from multiplexed DNA FISH data against known enhancer-promoter contacts.

## When to use

Use when SnapFISH loop summits have been called from chromatin tracing data and must be validated against known anchors (e.g., Sox2 promoter-super-enhancer) within a 10 Kb tolerance.

## When NOT to use

- Input is a count-based proximity-ligation assay dataset (e.g., bulk Hi-C); use HiCCUPS rather than SnapFISH output for anchor overlap checks
- Reference loop anchors are unknown or not genomically annotated; the 10 Kb overlap criterion requires pre-defined anchor coordinates
- Loop candidates have not yet been clustered into summits (Step 4a); singletons are removed and only summits should be checked against anchors

## Inputs

- SnapFISH output_loop_summit_<suffix>.txt (loop summit table with bin/genomic positions per allele)
- SnapFISH output_loop_candidate_<suffix>.txt (loop candidate table for candidate-count checks)
- ANN.txt 1D genomic annotation file (chr, start, end, pos per bin)
- Reference anchor coordinates (e.g., Sox2 promoter and super-enhancer positions in mm10)

## Outputs

- Boolean overlap call per allele: whether each loop summit anchor falls within 10 Kb of the reference promoter-super-enhancer anchors
- Validated promoter-super-enhancer loop presence/absence per haplotype (CAST and 129)
- Optional side-by-side comparison of SnapFISH summits vs HiCCUPS loops from mESC bulk Hi-C

## How to apply

After running 'SnapFISH call-loops' on each allele's 3D coordinates, inspect the output_loop_summit table and the 1D genomic annotation to obtain the genomic coordinates of the summit's two anchors. Define the reference anchors (e.g., the Sox2 promoter and its super-enhancer from the mESC Sox2 locus, mm10 chr3:34,601,078–34,806,078) and declare an overlap if and only if the SnapFISH loop anchors are within 10 Kb of the Sox2 promoter-super-enhancer loop anchors. Perform this check independently per allele (e.g., CAST vs 129), using distinct '-d' suffixes so outputs do not overwrite, and confirm each haplotype yields a single summit between the promoter and super-enhancer. Cross-reference with bulk Hi-C HiCCUPS loops as an orthogonal validation; the summit should be consistent with the HiCCUPS loop from mESC bulk Hi-C data.

## Related tools

- **SnapFISH** (Produces the loop summit and candidate tables whose anchors are checked for overlap with reference promoter-super-enhancer anchors within 10 Kb) — https://github.com/HuMingLab/SnapFISH
- **HiCCUPS** (Bulk Hi-C loop caller used as the orthogonal reference to confirm the SnapFISH summit is consistent with the known Sox2 enhancer-promoter loop)
- **pandas** (Loads and parses the output_loop_summit / output_loop_candidate text tables and the ANN.txt annotation for coordinate comparison)

## Examples

```
SnapFISH call-loops -i ext/129_3d_coor.txt -o tmp -a ext/input_ann.txt -p 1 -d 129
```

## Evaluation signals

- Loop summit anchors overlap the Sox2 promoter-super-enhancer anchors within 10 Kb in both CAST and 129 alleles
- Loop candidate counts match the expected values: 44 for CAST and 61 for 129 alleles
- Exactly a single loop summit falls between the Sox2 promoter and its super-enhancer per haplotype after candidate grouping
- The summit position is consistent with the HiCCUPS loop from mESC bulk Hi-C data
- Heatmaps of average Euclidean distance, contact frequency, loop candidates, and loop summits are produced when -p 1 is set

## Limitations

- The 10 Kb overlap criterion is a distance tolerance; summits slightly outside the anchors may represent true biological variation but will fail this check
- SnapFISH sensitivity can be further improved, in particular when applying to DNA seqFISH+ data in mESCs
- Only pair-wise chromatin interactions are considered; multi-way interactions are not detected

## Evidence

- [discussion] Overlapping Sox2 enhancer-promoter anchors: "if and only if the SnapFISH loop anchors are within 10 Kb of Sox2 promoter-super-enhancer loop anchors"
- [results] Summit detection in both haplotypes: [evidence span withheld — anchor status: not_found]
- [results] Candidate counts match expectation: [evidence span withheld — anchor status: not_found]
- [discussion] Summit selection by minimal FDR: [evidence span withheld — anchor status: not_found]
- [readme] Output files for verification: "tmp/output_loop_summit_129.txt, tmp/heatmap_loop_summits_129chr3.pdf: The loop summits are identified, and the heatmap showing their positions."
