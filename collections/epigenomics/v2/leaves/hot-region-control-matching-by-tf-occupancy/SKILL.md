---
name: hot-region-control-matching-by-tf-occupancy
description: Use when comparing a genomic signal track (e.g. G4-ChIP-seq IP vs input/control)
  over HOT regions and you need control intervals matched on the number of overlapping
  TF ChIP-seq peaks to separate feature-specific signal from hyper-ChIPable occupancy
  effects.
when_to_use_negative:
- When the question is whether TFs canonically bind HOT regions — the article shows
  HOT regions lack canonical motifs for TFs thought to be bound there, which requires
  motif analysis rather than occupancy-matched signal comparison
- When testing ChIP-seq artifact hypotheses via knocked-out TF data — false positive
  signals are demonstrated using ChIP-seq data sets for knocked-out transcription
  factors, not G4/control matching
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: bedtools
  role: coverage/intersect of G4-ChIP-seq IP and control tracks against HOT regions
    and occupancy-binned control intervals to get per-interval read coverage
- name: Python
  role: normalization by library depth, log2 enrichment computation, and aggregation
- name: pandas
  role: per-region enrichment table construction and group summary output
- name: scipy
  role: statistical comparison of log2 enrichment distributions for HOT vs occupancy-matched
    control groups
- name: matplotlib
  role: plotting the enrichment comparison between HOT regions and matched controls
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:2d5f27c59698ccc679812bc1310c5b387f424750fd1d852b2a769c08dda550f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - bedtools
  - matplotlib
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# hot-region-control-matching-by-tf-occupancy

## Summary

Builds TF-occupancy-matched control region sets so that enrichments measured on HOT regions (e.g. G4-ChIP-seq G-quadruplex signals) are not confounded by the sheer number of overlapping ChIP-seq peaks. This is the key confound-control step in the HOT-or-not analysis, since HOT regions are defined by unusually high TF ChIP-seq occupancy and may be ChIP-seq artifacts.

## When to use

Use when comparing a genomic signal track (e.g. G4-ChIP-seq IP vs input/control) over HOT regions and you need control intervals matched on the number of overlapping TF ChIP-seq peaks to separate feature-specific signal from hyper-ChIPable occupancy effects.

## When NOT to use

- When the question is whether TFs canonically bind HOT regions — the article shows HOT regions lack canonical motifs for TFs thought to be bound there, which requires motif analysis rather than occupancy-matched signal comparison
- When testing ChIP-seq artifact hypotheses via knocked-out TF data — false positive signals are demonstrated using ChIP-seq data sets for knocked-out transcription factors, not G4/control matching

## Inputs

- G4-ChIP-seq IP signal file (BAM or bigWig)
- G4-ChIP-seq input/control signal file (BAM or bigWig)
- HOT region coordinate set (BED)
- TF ChIP-seq peak sets used for occupancy binning

## Outputs

- Per-region log2(IP/control) enrichment table for HOT regions and occupancy-matched controls (g4_hot_enrichment.tsv)
- Group summary statistics (mean log2 enrichment per group) and scipy.stats comparison of the two distributions

## How to apply

First download the G4-ChIP-seq IP and input/control signal files (BAM/bigWig) used in the study and load the HOT region coordinate set. Build the control set by binning genomic intervals on the number of overlapping TF ChIP-seq peaks and selecting intervals matched to HOT regions within the same occupancy bins, so HOT-vs-control comparisons are not driven by peak count alone. Use bedtools coverage/intersect to obtain per-interval read coverage from the G4 IP and control tracks over HOT regions and the occupancy-matched controls, then normalize by library depth and compute log2(IP coverage / control coverage) per region. Aggregate log2 enrichment per group (HOT vs occupancy-matched controls) and compare the two distributions with scipy.stats; the article's motivation is that artificial ChIP-seq enrichment on HOT regions could be associated with GC/CpG-rich k-mers, R-loops, and G-quadruplex DNA.

## Related tools

- **bedtools** (coverage/intersect of G4-ChIP-seq IP and control tracks against HOT regions and occupancy-binned control intervals to get per-interval read coverage)
- **Python** (normalization by library depth, log2 enrichment computation, and aggregation)
- **pandas** (per-region enrichment table construction and group summary output)
- **scipy** (statistical comparison of log2 enrichment distributions for HOT vs occupancy-matched control groups)
- **matplotlib** (plotting the enrichment comparison between HOT regions and matched controls)

## Examples

```
bedtools coverage -a hot_regions.bed -b g4_ip.bam > g4_ip_cov.txt; bedtools coverage -a controls_matched.bed -b g4_input.bam > ctrl_cov.txt
```

## Evaluation signals

- Control regions occupy the same TF ChIP-seq peak-count bins as HOT regions (occupancy-matched by construction)
- Log2(IP/control) values are computed per region after library-depth normalization for both IP and control coverage
- Output table g4_hot_enrichment.tsv contains one log2 enrichment value per HOT region and per control region plus group summaries
- scipy.stats comparison of the two distributions yields a test statistic and p-value reported alongside mean enrichment per group
- Enrichment of G4-DNA on HOT regions should be interpretable in the context of R-loop localization on HOT regions

## Limitations

- The article argues HOT regions are likely to be ChIP-seq artifacts, so apparent enrichment may reflect hyper-ChIPable sequence features (GC/CpG-rich k-mers, R-loops, G-quadruplex DNA) rather than true TF co-binding
- A lack of signal in a KO ChIP-seq experiment might also be caused by technical conditions such as low number of reads, low library complexity or unsuccessful IP
- With currently available data it is not possible to estimate the proportion of antibody-specific error from pull-down of non-target proteins vs. direct binding to polynucleotide epitopes

## Evidence

- [other] Load the HOT region coordinate set and build the control region set by selecting genomic intervals matched to HOT regions on the number of overlapping TF ChIP-seq peaks (TF-occupancy binning).: [evidence span withheld — anchor status: not_found]
- [other] Normalize coverage by library depth and compute log2(IP coverage / control coverage) for each HOT region and each control region.: [evidence span withheld — anchor status: not_found]
- [abstract] We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [discussion] a lack of signal in a KO ChIP-seq experiment might also be caused by technical conditions such as low number of reads, low library complexity or unsuccessful IP.: "a lack of signal in a KO ChIP-seq experiment might also be caused by technical conditions such as low number of reads, low library complexity or unsuccessful IP."
