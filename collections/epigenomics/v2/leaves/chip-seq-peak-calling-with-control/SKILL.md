---
name: chip-seq-peak-calling-with-control
description: Use when starting from aligned-read BAM files of one or more ChIP-seq
  marks plus a matched input/control BAM, and you need enriched peak regions (e.g.
  as BED input for unsupervised alignment/clustering) rather than gene-anchored windows.
when_to_use_negative:
- When peaks are already available or the input is a pre-computed enrichment/feature
  table — only the subsequent DGW stages are needed, not peak calling.
- When the analysis is anchored to genomic landmarks such as transcription start sites
  or gene annotation, since this skill specifically derives peak regions from MACS2
  calls rather than annotation-anchored windows.
- When no matched input/control sample exists for the ChIP-seq experiment, as MACS2
  enrichment here is defined relative to the input control BAM.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0160
tools:
- name: MACS2
  role: Peak caller used to identify regions of ChIP-seq enrichment relative to the
    input control sample from aligned-fragment BAM files
- name: MACS
  role: Generic peak finder whose output peak regions provide the input enrichment
    measurements for downstream DGW analysis
- name: DGW
  role: Downstream consumer of the filtered merged peak BED; its bin-size (50 bp)
    and min-pileup requirements determine the peak filtering thresholds
  repo: github.com/lukauskas/dgw
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:1b6af3adad424066a5f121164031dcf3e45ccf68cda6e1745d13bd8d501cc4d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGW
  - MACS
  - MACS2
  - Python
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# chip-seq-peak-calling-with-control

## Summary

Call ChIP-seq enriched regions with MACS2 against an input control sample (e.g. ENCODE K562 H3K4me3 and H3K9ac Std AlnRep1 BAMs vs the K562 Control Std AlnRep1 BAM) and merge peak sets across marks to define regions of interest for downstream shape-based analysis such as DGW.

## When to use

Use when starting from aligned-read BAM files of one or more ChIP-seq marks plus a matched input/control BAM, and you need enriched peak regions (e.g. as BED input for unsupervised alignment/clustering) rather than gene-anchored windows.

## When NOT to use

- When peaks are already available or the input is a pre-computed enrichment/feature table — only the subsequent DGW stages are needed, not peak calling.
- When the analysis is anchored to genomic landmarks such as transcription start sites or gene annotation, since this skill specifically derives peak regions from MACS2 calls rather than annotation-anchored windows.
- When no matched input/control sample exists for the ChIP-seq experiment, as MACS2 enrichment here is defined relative to the input control BAM.

## Inputs

- Aligned-read BAM files for each ChIP-seq mark (e.g. wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam, wgEncodeBroadHistoneK562H3k9acStdAlnRep1.bam) with .bam.bai index files
- Matched input control BAM (e.g. wgEncodeBroadHistoneK562ControlStdAlnRep1.bam) with .bam.bai index files
- Bin size defining the length filter scale (50 bp non-overlapping windows used in the DGW workflow)

## Outputs

- Per-mark MACS2 peak call sets (enriched regions relative to input control)
- Merged BED file containing the union of regions called for either mark
- Filtered BED file of peaks >5 and <1000 bins with at least 10 counts, ready for dgw-worker

## How to apply

Run MACS2 callpeak separately for each epigenomic mark, treating the aligned-fragment BAM (e.g. wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam) as the treatment and the input control BAM (wgEncodeBroadHistoneK562ControlStdAlnRep1.bam) as the background, to identify regions enriched relative to the control. Then merge the per-mark peak sets by taking the union of every region called for either mark, so downstream analysis uses a single shared region set across marks. Restrict the merged set to peaks longer than 5 and shorter than 1000 bins (at 50 bp bin size, i.e. >250 bp and <50 kb) and with at least 10 counts, and drop regions with no bin holding more than 10 reads (the -min-pileup behaviour), since DGW cannot meaningfully align empty profiles. The rationale is that DGW operates on ChIP-seq enrichment measurements within MACS-called peak regions and requires regions with sufficient signal to produce informative binning and DTW distances.

## Related tools

- **MACS2** (Peak caller used to identify regions of ChIP-seq enrichment relative to the input control sample from aligned-fragment BAM files)
- **MACS** (Generic peak finder whose output peak regions provide the input enrichment measurements for downstream DGW analysis)
- **DGW** (Downstream consumer of the filtered merged peak BED; its bin-size (50 bp) and min-pileup requirements determine the peak filtering thresholds) — github.com/lukauskas/dgw

## Examples

```
macs2 callpeak -t wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam -c wgEncodeBroadHistoneK562ControlStdAlnRep1.bam -n K562_H3K4me3
```

## Evaluation signals

- MACS2 completes for every mark without error and produces a peak set for each treatment-vs-control comparison.
- The merged BED contains the union of regions called for either mark, with no mark's peaks missing from the merged set.
- All peaks in the final BED are longer than 5 and shorter than 1000 bins (with 50 bp bins) and have at least 10 counts.
- dgw-worker accepts the filtered BED with the two BAM datasets, confirming region lengths and pileup satisfy DGW's binning requirements.

## Limitations

- Peaks shorter than 5 bins or longer than 1000 bins are excluded, so very sharp or very broad enrichment regions fall outside the analysis.
- Regions with no bin containing more than 10 reads (unexpressed regions) are ignored, which can silently drop weakly enriched peaks.
- Merging by union of regions called for either mark means a region enriched for only one mark is analysed with a flat/empty track for the other mark.

## Evidence

- [methods] Aligned fragments (BAM files) of both epigenomic marks were processed with the MACS2 peak caller: "Aligned fragments (BAM files) of both epigenomic marks were processed with the MACS2 peak caller [10] to identify regions which showed enrichment relative to a input control sample"
- [methods] we then merged the two sets by considering every region called for either mark: "we then merged the two sets by considering every region called for either mark"
- [methods] restricted the analysed set of peaks to those that had a length larger than 5 and smaller than 1000 bins: "restricted the analysed set of peaks to those that had a length larger than 5 and smaller than 1000 bins"
- [methods] the unexpressed regions that have no bin with more than 10 reads in it (`-min-pileup` constraint to change) will be ignored: "the unexpressed regions that have no bin with more than 10 reads in it (`-min-pileup` constraint to change) will be ignored"
- [intro] as input data we use ChIP-Seq enrichment measurements within peak regions identified by a peak: "as input data we use ChIP-Seq enrichment measurements within peak regions identified by a peak finder such as MACS"
