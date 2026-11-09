---
name: genomic-mappability-and-blacklist-filtering
description: Use when running SnapHiC (or similar loop-calling pipelines) on scHi-C/sn-m3C-seq
  contact files and you must restrict loop candidate calling to bins with adequate
  mappability, excluding ENCODE blacklist overlaps.
when_to_use_negative:
- Your analysis is not bin-based Hi-C loop calling — e.g., RNA-seq quantification
  or 3D genome reconstruction with Dip-C, where blacklist filtering serves a different
  purpose and must be applied separately
- Your bin resolution or restriction enzyme differs from the provided files (e.g.,
  not 10-kb with MboI) and you have not downloaded the matching mappability features
  — reusing mismatched filter files would exclude the wrong bins
- The genome build of the contact files does not match the filter file's build (mm10
  vs hg19 vs hg38), which would misalign exclusions
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SnapHiC
  role: Consumes the filter-regions BED file via the filter_file variable to exclude
    low-mappability and ENCODE blacklist bins from loop candidate identification during
    the 'interaction' step
  repo: https://github.com/HuMingLab/SnapHiC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:705a2f18795ba0d0808a4975e2974bb49add6b890b19625e684126e51d9ce217
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Java >= 8
  - MPI (open-mpi)
  - Python >= 3.6
  - SnapHiC
  - cooler
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# genomic-mappability-and-blacklist-filtering

## Summary

Excludes low-mappability genomic bins and ENCODE blacklist regions from single-cell Hi-C loop calling by supplying a pre-binned filter-regions BED file (e.g., 10-kb bins with mappability <=0.8 under the MboI restriction enzyme) to SnapHiC, preventing spurious loop candidates in poorly mappable or artifact-prone regions.

## When to use

Use when running SnapHiC (or similar loop-calling pipelines) on scHi-C/sn-m3C-seq contact files and you must restrict loop candidate calling to bins with adequate mappability, excluding ENCODE blacklist overlaps.

## When NOT to use

- Your analysis is not bin-based Hi-C loop calling — e.g., RNA-seq quantification or 3D genome reconstruction with Dip-C, where blacklist filtering serves a different purpose and must be applied separately
- Your bin resolution or restriction enzyme differs from the provided files (e.g., not 10-kb with MboI) and you have not downloaded the matching mappability features — reusing mismatched filter files would exclude the wrong bins
- The genome build of the contact files does not match the filter file's build (mm10 vs hg19 vs hg38), which would misalign exclusions

## Inputs

- Binned BED file of excluded genomic regions (bins with mappability <=0.8 plus ENCODE blacklist overlaps), e.g., ext/hg19_filter_regions.txt at 10-kb resolution with MboI
- chrom.sizes file for the reference genome (mm10, hg19, or hg38), e.g., ext/hg19.chrom.sizes
- Per-cell tab-separated contact files (2 columns of chromosome names and 2 columns of mapped positions in bp)

## Outputs

- Filter_file configuration consumed by SnapHiC run_step1.sh, restricting loop candidate identification to mappable, non-blacklisted 10-kb bins
- Downstream *.postprocessed.summits.bedpe loop calls (17 columns including outlier_count and fdr_dist) free of excluded-region artifacts

## How to apply

Obtain or build a binned BED file of excluded regions: define mappability per 10-kb bin based on the MboI restriction enzyme, remove all bins with mappability <=0.8, and remove all bins overlapped with ENCODE blacklist regions. For mm10, hg19, and hg38 at 10-kb resolution with MboI, use the precomputed filter-region files shipped in the SnapHiC `ext` directory (e.g., ext/hg19_filter_regions.txt); for other genomes, resolutions, or restriction enzymes, download local genomic features including mappability scores from the renlab resource URL. Point the `filter_file` variable in run_step1.sh to this file alongside the matching chrom.sizes file, so that the 'interaction' step (identifying loop candidates against global and local background models) never calls loops in excluded bins. Rationale: unmappable sequence and blacklist regions produce artifactual read piles that otherwise masquerade as loop candidates.

## Related tools

- **SnapHiC** (Consumes the filter-regions BED file via the filter_file variable to exclude low-mappability and ENCODE blacklist bins from loop candidate identification during the 'interaction' step) — https://github.com/HuMingLab/SnapHiC

## Examples

```
filter_file="ext/hg19_filter_regions.txt"  # set in run_step1.sh; supplied to snap.py --steps "interaction" to exclude mappability<=0.8 and ENCODE blacklist bins
```

## Evaluation signals

- The filter file contains only binned BED intervals restricted to the chosen genome (hg19) and resolution (10-kb), combining mappability <=0.8 bins with ENCODE blacklist overlaps
- No loop summits in the *.postprocessed.summits.bedpe output fall within bins listed in the filter file
- Per-cell-type loop counts remain within the expected published range (approximately 817 to 27,379 loops at 10-kb resolution across the 14 prefrontal cortex cell types)
- Output bedpe files retain all 17 required columns, including outlier_count and fdr_dist, confirming filtering did not corrupt the postprocessed results

## Limitations

- Prebuilt filter files are only provided for mm10, hg19, and hg38 at 10-kb resolution with the MboI restriction enzyme; other configurations require downloading external mappability/local genomic feature resources
- The mappability definition is tied to a specific previous study's enzyme-based model (PMID: 23023982), so mappability scores are MboI-specific and may not transfer to other restriction enzymes
- Filtering excludes entire bins rather than rescoring them; genuine signals overlapping blacklist or low-mappability bins are lost by design

## Evidence

- [methods] Mappability-based and blacklist bin exclusion at 10 kb with MboI: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] Precomputed filter-region files for three genome builds: "Filtered regions for mm10, hg19 and hg38 at 10KB resolution with the restriction enzyme MboI, i.e., bins with mappability <=0.8 and bins overlapped with the ENCODE blacklist regions, are included in"
- [readme] How the filter file is wired into SnapHiC execution: "`filter_file`="ext/mm10_filter_regions.txt" (A binned bed file of the genomic regions that are excluded from loop calling, such as the low mappability regions and the ENCODE blacklist regions."
