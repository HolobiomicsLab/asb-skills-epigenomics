---
name: narrowpeak-format-parsing
description: Use when the input consists of narrowPeak ChIP-seq peak files and you
  need peak summit coordinates to build a genome-wide TF-occupancy track or define
  HOT regions.
when_to_use_negative:
- Input peaks are in a different format (e.g. broadPeak or bedGraph signal tracks)
  that lacks the narrowPeak 'peak' summit-offset column — the summit extraction formula
  chromStart + peak offset would be invalid.
- The analysis goal is something other than TF-occupancy/HOT-region definition, e.g.
  calling individual TF binding peaks de novo from FASTQ/BAM data — this skill assumes
  peaks have already been called.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Python
  role: Used with pandas/pybedtools to load narrowPeak files and extract peak summit
    coordinates for each TF dataset per species
- name: pybedtools
  role: 'Interval arithmetic on narrowPeak entries: extracting summits and tallying
    distinct summits into sliding windows tiling each genome assembly'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:c944472208b7894d7b898755a2619369ad725cbc4934e9395b3a15f83af95c18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - pybedtools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# narrowpeak-format-parsing

## Summary

Parses ENCODE/modENCODE narrowPeak ChIP-seq peak files (hg19, mm9, dm3, ce10) and extracts per-TF peak summit coordinates (chromStart + peak offset column) so that TF-occupancy counts per genomic window can be tallied and HOT regions called. This is the data-ingestion step of the Wreczycka et al. 2019 HOT-region pipeline.

## When to use

Use when the input consists of narrowPeak ChIP-seq peak files and you need peak summit coordinates to build a genome-wide TF-occupancy track or define HOT regions.

## When NOT to use

- Input peaks are in a different format (e.g. broadPeak or bedGraph signal tracks) that lacks the narrowPeak 'peak' summit-offset column — the summit extraction formula chromStart + peak offset would be invalid.
- The analysis goal is something other than TF-occupancy/HOT-region definition, e.g. calling individual TF binding peaks de novo from FASTQ/BAM data — this skill assumes peaks have already been called.

## Inputs

- narrowPeak ChIP-seq peak files from ENCODE/modENCODE for each TF dataset (hg19, mm9, dm3, ce10 assemblies)
- Genome assembly window tiling / chromosome sizes for hg19, mm9, dm3, ce10

## Outputs

- Per-TF peak summit coordinate table (chromosome, summit position) per species
- Per-window TF-occupancy count track over each genome assembly
- BED tracks of HOT regions per species (e.g. hot_regions_hg19.bed, hot_regions_mm9.bed, hot_regions_dm3.bed, hot_regions_ce10.bed)

## How to apply

Load the narrowPeak peak files (tab-delimited, no header) for each genome assembly (hg19, mm9, dm3, ce10) with Python using pandas/pybedtools. Extract the peak summit coordinate from each narrowPeak entry as chromStart (chromStart column) plus the 'peak' offset column (the summit offset, 9th 0-based field) for every TF dataset per species. Tally the number of distinct peak summits falling into consecutive sliding windows tiling each genome assembly to build a per-window TF-occupancy count track; downstream, the HOT-or-not analysis calls windows at or above the 99th percentile of the genome-wide occupancy distribution as HOT regions and merges adjacent qualifying windows. Correctness is judged by writing the resulting HOT regions as BED tracks (hot_regions_hg19.bed, hot_regions_mm9.bed, hot_regions_dm3.bed, hot_regions_ce10.bed) and comparing counts against the reported 4324 (human), 2638 (mouse), 422 (C. elegans) and 408 (D. melanogaster) HOT regions.

## Related tools

- **Python** (Used with pandas/pybedtools to load narrowPeak files and extract peak summit coordinates for each TF dataset per species)
- **pybedtools** (Interval arithmetic on narrowPeak entries: extracting summits and tallying distinct summits into sliding windows tiling each genome assembly)

## Examples

```
import pandas as pd; peaks = pd.read_csv('peaks.narrowPeak', sep='\t', header=None); peaks['summit'] = peaks[1] + peaks[9]
```

## Evaluation signals

- Summit coordinate invariant: every extracted summit must equal the narrowPeak chromStart plus the value of the 9th (0-based) 'peak' offset column.
- The number of qualifying windows after applying the 99th-percentile TF-occupancy threshold, merged, should reproduce the reported species counts: 4324 (human/hg19), 2638 (mouse/mm9), 422 (C. elegans/ce10), 408 (D. melanogaster/dm3).
- Output BED tracks (hot_regions_hg19.bed etc.) must contain valid coordinates within the corresponding assembly's chromosome sizes and no overlapping intervals after merging adjacent qualifying windows.
- Each TF dataset contributes at least one summit to the occupancy track; a TF whose narrowPeak file parses to zero summits indicates a parsing failure (wrong column indexing or malformed file).

## Limitations

- The HOT regions defined by this pipeline are, per the article's own conclusion, likely ChIP-seq artifacts similar to 'hyper-ChIPable' regions, so parsed HOT tracks should be interpreted with caution rather than as true multi-TF binding sites.
- Lack of signal in knockout (KO) ChIP-seq experiments may stem from technical conditions such as low number of reads, low library complexity or unsuccessful IP, affecting downstream interpretation of occupancy tracks.
- The public repository README only documents the analysis script Scripts/HOT-region-main.R; no changelog artifact is available, so reproducibility of parsing parameters relies on the manuscript description.

## Evidence

- [intro] Large scale projects such as ENCODE (1) and modENCODE (2) used this technology to find the binding sites of hundreds of proteins in multiple species.: "Large scale projects such as ENCODE (1) and modENCODE (2) used this technology to find the binding sites of hundreds of proteins in multiple species."
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
- [abstract] Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions.: "Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions."
