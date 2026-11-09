---
name: genome-chromosome-coordinate-segmentation
description: Use when multiple ChIP-seq samples with spike-in chromatin must be compared
  genome-wide and a common fixed-size bin grid (e.g. 1000-bp bins of hg19 and FLY-suffixed
  dm3 from ChromInfo.txt) is needed before per-bin counting and normalization.
when_to_use_negative:
- The analysis targets a pre-selected subset of loci or peak intervals rather than
  genome-wide binned signal — spikChIP normalizes over the complete genome and its
  bin grid is designed for that scope
- No ChromInfo chromosome-size file exists for the assembly, since spikChIP derives
  the binning grid directly from this list
- Downstream analysis already consumes a feature/count table at another resolution
  and does not need re-segmentation into fixed bins
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3169
tools:
- name: spikChIP
  role: Reads the configuration and ChromInfo files and performs the segmentation
    of the sample and spike-in genomes into bins of size set by -k before counting
    and normalizing
  repo: https://github.com/eblancoga/spikChIP
- name: SeqCode
  role: Counts reads of the BAM files overlapping the 1000-bp bins and peaks of the
    BED files after segmentation
  repo: https://github.com/eblancoga/seqcode
- name: Samtools
  role: Informs about the number of reads of the BAM files processed against the segmented
    bins
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:39c2a8a4d0413fcf3b193d953d1bef32c57c5478a9f3f4d04c071d75278a2af1
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
  - R
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# genome-chromosome-coordinate-segmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Segments the sample (e.g. hg19) and spike-in (e.g. dm3) genomes into fixed-size genomic bins (1000 bp in the H3K79me2 Jurkat series) driven by a ChromInfo chromosome list, creating the coordinate grid on which per-bin read counts and spikChIP normalization are computed.

## When to use

Use when multiple ChIP-seq samples with spike-in chromatin must be compared genome-wide and a common fixed-size bin grid (e.g. 1000-bp bins of hg19 and FLY-suffixed dm3 from ChromInfo.txt) is needed before per-bin counting and normalization.

## When NOT to use

- The analysis targets a pre-selected subset of loci or peak intervals rather than genome-wide binned signal — spikChIP normalizes over the complete genome and its bin grid is designed for that scope
- No ChromInfo chromosome-size file exists for the assembly, since spikChIP derives the binning grid directly from this list
- Downstream analysis already consumes a feature/count table at another resolution and does not need re-segmentation into fixed bins

## Inputs

- ChromInfo.txt plain-text file with chromosome names and sizes for the sample (hg19) and spike-in (dm3, 'FLY'-suffixed) genomes
- BAM files of sample and spike-in genomes mapped with Bowtie (multi-locus reads discarded), one sample/spike pair per condition (GSM1465004-GSM1465008)
- MACS2 broad-peak BED files for sample and spike-in genomes (used later to label bins as peaks or background)
- Bin size parameter in kbp (e.g. -k 1 for 1000-bp bins)

## Outputs

- Fixed-size (1000-bp) bin segmentation of both the sample and spike-in genomes
- Per-bin read-count tables (average and maximum values) in the results/ folder, e.g. FINAL_H3K79me2_0-25-50-75-100_SPIKCHIP_1000_avg_normalized_sample.txt.gz
- Bins labeled as peaks or background, feeding the boxplot comparisons in plots/ (PDF)

## How to apply

Read the ChromInfo plain-text file listing chromosome names and sizes for both the sample genome (hg19) and the spike-in genome (dm3 chromosomes carrying the 'FLY' suffix), and verify all input BAM/BED files exist before segmenting. Segment both genomes into bins of the SAME size — 1000 bp for the GSE60104 Jurkat H3K79me2 five-sample analysis, set via the -k parameter expressed in kbp — so that bins are directly comparable between human and fruit fly. Each bin inherits its coordinates from the chromosome list; the bin size is a tunable decision point (e.g. -k 1) but must be identical for sample and spike-in. The binned coordinates then serve as the unit for counting reads inside the bins (average and maximum values with SeqCode/Samtools) and for classifying bins as peaks or background using the user-provided MACS2 broad-peak BED files, after which the SPIKCHIP, RAW, TRADITIONAL, CHIPRX and TAG REMOVAL normalizations are computed per bin.

## Related tools

- **spikChIP** (Reads the configuration and ChromInfo files and performs the segmentation of the sample and spike-in genomes into bins of size set by -k before counting and normalizing) — https://github.com/eblancoga/spikChIP
- **SeqCode** (Counts reads of the BAM files overlapping the 1000-bp bins and peaks of the BED files after segmentation) — https://github.com/eblancoga/seqcode
- **Samtools** (Informs about the number of reads of the BAM files processed against the segmented bins)

## Examples

```
./spikChIP -vhcde FLY -k 1 -p 1 config/H3K79me2_0-25-50-75-100_config.txt ChromInfo.txt
```

## Evaluation signals

- The bin size in output tables matches the -k parameter (1000 bp, i.e. -k 1 in kbp) and is identical for the sample and spike-in genomes
- Every chromosome in the ChromInfo list (hg19 plus FLY-suffixed dm3 contigs) yields bins covering its full length with no gaps
- Per-bin count rows align 1:1 between the human sample table and the spike-in table structure for each condition
- Bins classified as peaks via MACS2 broad-peak BED overlap sum to a plausible subset of total bins, with the remaining bins forming flat background distributions across all five conditions
- The final SPIKCHIP-normalized file (e.g. FINAL_H3K79me2_0-25-50-75-100_SPIKCHIP_1000_avg_normalized_sample.txt.gz) exists in results/ and the run log logs/H3K79me2_0-25-50-75-100.log reports no missing BAM/BED files

## Limitations

- Both genomes must be segmented into bins of the same size; mismatched bin sizes between sample and spike-in break the correspondence
- BAM files with < 1 million reads require the -d option to be processed at all
- Upstream mapping discards multi-locus reads (Bowtie), so bins in repetitive regions may be undercounted
- Peak/background classification of bins depends entirely on the quality of the user-supplied MACS2 broad-peak BED files

## Evidence
