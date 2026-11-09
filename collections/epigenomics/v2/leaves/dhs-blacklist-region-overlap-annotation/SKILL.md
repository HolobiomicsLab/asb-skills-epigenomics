---
name: dhs-blacklist-region-overlap-annotation
description: Decision support for evaluating whether called ChIP-seq/DNase-seq peaks
  fall in biologically credible regulatory space (DHS) versus artifact-prone blacklist
  regions during QC.
when_to_use_negative:
- Input peaks are per-replicate only and no merged peak file exists — this check is
  performed on merged peaks after step 6 of ChiLin
- Reference genome lacks downloadable dependent data (no DHS/blacklist tracks available
  from Cistrome DB for the assembly)
- The goal is replicate concordance (use wigCorrelate Pearson correlation and peak-overlap
  ratio instead)
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: ChiLin
  role: Orchestrates the DHS/blacklist overlap QC step and compiles id.dhs statistics
    into the QC summary
  repo: https://github.com/cfce/chilin
- name: bedtools
  role: Intersects merged peak files with union DHS and blacklist (velcro) region
    BED files
- name: MACS2
  role: Produces the merged narrowPeak/broadPeak peak files that are evaluated against
    DHS and blacklist regions
  repo: https://github.com/taoliu/MACS/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:9689db2cef609b65ad14e2747f22675d674a1f1795d49836781a27dccdb4590c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - MACS2
  - bedClip
  - bedGraphToBigWig
  - bedtools
  - samtools
  - wigCorrelate
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# dhs-blacklist-region-overlap-annotation

## Summary

Assess merged ChIP-seq peak quality by measuring overlap with union DHS regions and blacklist (velcro) regions using bedtools, as implemented in ChiLin's QC layer to produce the id.dhs overlap statistics.

## When to use

Decision support for evaluating whether called ChIP-seq/DNase-seq peaks fall in biologically credible regulatory space (DHS) versus artifact-prone blacklist regions during QC.

## When NOT to use

- Input peaks are per-replicate only and no merged peak file exists — this check is performed on merged peaks after step 6 of ChiLin
- Reference genome lacks downloadable dependent data (no DHS/blacklist tracks available from Cistrome DB for the assembly)
- The goal is replicate concordance (use wigCorrelate Pearson correlation and peak-overlap ratio instead)

## Inputs

- Merged MACS2 peak file (id_peaks.narrowPeak or broadPeak) from combined replicate BAMs
- Union DHS regions BED file for the reference genome (hg19/hg38/mm9/mm10 from Cistrome DB dependent data)
- Blacklist (velcro) regions BED file from Cistrome DB dependent data
- bedtools for interval intersection

## Outputs

- id.dhs file with DHS and blacklist overlap statistics for merged peaks
- QC summary entries reporting DHS overlap and blacklist overlap fractions

## How to apply

After MACS2 peak calling on merged replicate BAMs (narrowPeak for TFs/active marks, broadPeak for broad histone marks), intersect the merged peak file with union DHS regions derived from Cistrome DB and with blacklist (velcro) regions using bedtools. Compute the fraction of peaks overlapping DHS and the fraction overlapping blacklist regions as QC statistics, written by ChiLin as the id.dhs output. High DHS overlap indicates peaks localize to accessible regulatory DNA, while high blacklist overlap signals artifact-driven peaks that warrant re-examination of filtering or peak-calling parameters. These statistics are compiled alongside peak counts, FRiP, and replicate overlap/correlation values into the plain-text/JSON QC summary.

## Related tools

- **ChiLin** (Orchestrates the DHS/blacklist overlap QC step and compiles id.dhs statistics into the QC summary) — https://github.com/cfce/chilin
- **bedtools** (Intersects merged peak files with union DHS and blacklist (velcro) region BED files)
- **MACS2** (Produces the merged narrowPeak/broadPeak peak files that are evaluated against DHS and blacklist regions) — https://github.com/taoliu/MACS/

## Examples

```
bedtools intersect -a merged_peaks.narrowPeak -b union_dhs.bed -u | wc -l  # DHS overlap; repeat with velcro blacklist to populate id.dhs
```

## Evaluation signals

- Non-empty id.dhs output file reporting DHS overlap statistics for merged peaks
- Non-empty merged peak file (id_peaks.narrowPeak or broadPeak) prior to intersection
- DHS overlap fraction is consistent with the fraction of peaks in accessible regulatory regions expected for the factor class
- Blacklist overlap fraction is low; elevated values indicate artifact-prone peaks needing re-filtering
- Statistics are compiled into the plain-text/JSON QC summary alongside FRiP and replicate overlap values

## Limitations

- Depends on organism- and assembly-specific dependent data (hg19/hg38/mm9/mm10 DHS and velcro blacklist files downloaded from Cistrome DB); unavailable assemblies cannot be assessed
- Interpreted as a population-level QC signal rather than per-peak truth; overlap alone does not validate individual peak calls
- Statistical overlap behavior varies with peak count and broad vs narrow peak mode, so metrics are comparable mainly within the same assay class

## Evidence

- [abstract] ChiLin automates QC and analysis of ChIP-seq and DNase-seq data in a unified, modular framework: "ChiLin is a computational pipeline that automates the quality control and data analyses of ChIP-seq and DNase-seq data."
- [methods] Evaluate merged peak overlap with union DHS and blacklist regions: [evidence span withheld — anchor status: not_found]
- [intro] ChiLin uses MACS2 as the default peak caller: "ChiLin uses MACS2 (https://github.com/taoliu/MACS/) as the default peak caller"
