---
name: genomic-bin-segmentation
description: 'Use when preparing a spikChIP or similar spike-in ChIP-seq pipeline:
  before counting reads per bin or classifying bins as peaks/background, the sample
  and spike-in genomes must be divided into fixed-size bins from a ChromInfo chromosome
  list.'
when_to_use_negative:
- When analysis targets only a pre-defined subset of loci or already-provided peak
  intervals rather than the complete genome — spikChIP normalization is genome-wide
  over bins
- When peaks themselves define the analysis units directly and no per-bin read counting
  is needed downstream
- When the genome assemblies lack a complete ChromInfo chromosome length list, since
  bins follow the declared chromosome names and sizes
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0160
tools:
- name: spikChIP
  role: Perl pipeline whose -k option sets the bin size (in kbp) used to segment the
    sample and spike-in genomes during Step 1 of the run
  repo: https://github.com/eblancoga/spikChIP
- name: SeqCode
  role: Consumes the segmented bins to count reads of BAM files per bin and to overlap
    bins with peaks of BED files (matchpeaks) for peak/background labeling
  repo: https://github.com/eblancoga/seqcode
- name: MACS2
  role: Produces the broad-peak BED files whose coordinates are overlapped against
    the bins after segmentation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:f18f57c87f875c294dac8104195b66eb4666673bf69a516a16e305ca3e4d89b2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/eblancoga/spikChIP
  repo_url: https://github.com/eblancoga/spikChIP
  tools:
  - Bowtie
  - MACS2
  - SeqCode
  - spikChIP
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# genomic-bin-segmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Segmentation of the sample (e.g. hg19 human) and spike-in (e.g. dm3 fly) genomes into fixed-size bins (default 1,000 bp, tunable via the -k option of spikChIP) using the chromosome name/length list in ChromInfo.txt, so that ChIP-seq reads and MACS2-called peaks can be assigned per-bin in downstream normalization.

## When to use

Use when preparing a spikChIP or similar spike-in ChIP-seq pipeline: before counting reads per bin or classifying bins as peaks/background, the sample and spike-in genomes must be divided into fixed-size bins from a ChromInfo chromosome list.

## When NOT to use

- When analysis targets only a pre-defined subset of loci or already-provided peak intervals rather than the complete genome — spikChIP normalization is genome-wide over bins
- When peaks themselves define the analysis units directly and no per-bin read counting is needed downstream
- When the genome assemblies lack a complete ChromInfo chromosome length list, since bins follow the declared chromosome names and sizes

## Inputs

- ChromInfo.txt plain-text file listing chromosome names and lengths (hg19 sample and dm3 spike-in, 'FLY'-suffixed)
- Bin size parameter in kbp (spikChIP -k option; 1-kbp bins in the reference workflow)
- Genome assembly references for sample and spike-in genomes (e.g. hg19, dm3)

## Outputs

- Fixed-size (e.g. 1,000 bp) non-overlapping genomic bins for the sample genome (hg19)
- Matching fixed-size bins for the spike-in genome (dm3)
- Bin coordinate tables used for per-bin read counting and peak/background classification

## How to apply

Load the ChromInfo.txt file, a plain-text file listing chromosome names and lengths for both genomes (hg19 as sample, dm3 as 'FLY'-suffixed spike-in), and split every chromosome into non-overlapping fixed-size bins; for the spikChIP reference workflow, 1-kbp bins are used (-k 1, the bin size in kbp). Both the sample and the spike-in genome must be segmented into bins of the same size so bin-level read counts and peak overlaps are comparable. The resulting bin coordinates are the substrate for read counting (average and maximum per bin) with SeqCode and for classifying bins as peaks or background via overlap with MACS2 broad peaks (e.g. peak_files/1_sample_peaks.bed from GSM1465005/Jurkat_K79_25%_R1). Rationale: per-bin segmentation enables spikChIP's local-regression normalization to apply the spike-in correction increasingly from background to ChIP-enriched regions rather than uniformly over the whole genome.

## Related tools

- **spikChIP** (Perl pipeline whose -k option sets the bin size (in kbp) used to segment the sample and spike-in genomes during Step 1 of the run) — https://github.com/eblancoga/spikChIP
- **SeqCode** (Consumes the segmented bins to count reads of BAM files per bin and to overlap bins with peaks of BED files (matchpeaks) for peak/background labeling) — https://github.com/eblancoga/seqcode
- **MACS2** (Produces the broad-peak BED files whose coordinates are overlapped against the bins after segmentation)

## Examples

```
./spikChIP -vhcde FLY -k 1 -p 0 config_file.txt hg19_dm3_chrominfo.txt
```

## Evaluation signals

- Bin count per chromosome equals ceiling(chromosome length / bin size) computed from ChromInfo.txt lengths (1-kbp bins in the reference run)
- Sample (hg19) and spike-in (dm3) bin sets are segmented at the same bin size as declared via -k
- Downstream labelled bin files contain non-null bins assigned to both the peaks and background classes
- Bin coordinates are valid BED intervals within declared chromosome lengths (no bins exceeding chromosome ends)
- Output BED/bin files exist in valid BED format matching the published spikChIP results

## Limitations

- Segmentation depends entirely on the chromosome list declared in ChromInfo.txt; missing or incorrect chromosome entries silently drop bins
- Bin classification (peaks vs background) depends on peaks provided by the user; poor peak calling with MACS2 broad-peak mode propagates into mislabeled bins
- Bins of < 1 Million reads in BAM handling require the -d option, affecting downstream per-bin counts for small spike-in genomes

## Evidence
