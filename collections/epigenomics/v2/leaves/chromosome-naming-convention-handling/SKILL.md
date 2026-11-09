---
name: chromosome-naming-convention-handling
description: Use when preparing a spike-in ChIP-seq normalization run (e.g. spikChIP
  with human hg19 sample plus dm3 fly spike-in) and a chromInfo file or combined reference
  genome must be assembled with renamed exogenous chromosomes.
when_to_use_negative:
- The analysis is single-genome ChIP-seq without spike-in — no exogenous chromosome
  renaming is needed and spikChIP's FLY/HUMAN/MOUSE/EXO distinction does not apply
- The BAM files already use the required exogenous genome identifier (e.g. FLY prefix)
  and a valid combined ChromInfo file exists
- Peak calling or downstream normalization steps (MACS2, spikChIP normalization) are
  being run on pre-existing correctly named files — this skill belongs to the pre-mapping
  setup stage
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: spikChIP
  role: Consumes the ChromInfo file (second positional argument) and enforces the
    exogenous genome identifier via -e (default FLY, allowed FLY,HUMAN,MOUSE,EXO)
    to separate sample and spike-in bins
  repo: https://github.com/eblancoga/spikChIP
- name: Bowtie
  role: Indexes the concatenated hg19+dm3 synthetic genome so that fly reads align
    to FLY-suffixed chromosomes and can be filtered out later
  repo: http://bowtie-bio.sourceforge.net
- name: SeqCode
  role: Segments both genomes into non-overlapping bins (e.g. 1-kbp) using the ChromInfo
    file and counts reads per bin
  repo: https://github.com/eblancoga/seqcode
- name: Samtools
  role: Filters unmapped reads (-F 0x4) and supports splitting the filtered BAM into
    human and FLY spike-in read sets
- name: gawk
  role: Formats per-bin count tables keyed by chromosome names from the ChromInfo
    convention
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:1bd9f184b0f1aa72eb706b477bb0465fd91b5cf9ae4ccebff0e571e98152c2a4
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
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# chromosome-naming-convention-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the combined sample+spike-in ChromInfo file required by spikChIP by renaming exogenous (dm3 fruit fly) chromosomes with the mandatory 'FLY' suffix and merging them with sample (hg19 human) chromosomes, so that a single synthetic genome index and bin segmentation can distinguish sample from spike-in reads.

## When to use

Use when preparing a spike-in ChIP-seq normalization run (e.g. spikChIP with human hg19 sample plus dm3 fly spike-in) and a chromInfo file or combined reference genome must be assembled with renamed exogenous chromosomes.

## When NOT to use

- The analysis is single-genome ChIP-seq without spike-in — no exogenous chromosome renaming is needed and spikChIP's FLY/HUMAN/MOUSE/EXO distinction does not apply
- The BAM files already use the required exogenous genome identifier (e.g. FLY prefix) and a valid combined ChromInfo file exists
- Peak calling or downstream normalization steps (MACS2, spikChIP normalization) are being run on pre-existing correctly named files — this skill belongs to the pre-mapping setup stage

## Inputs

- hg19 (sample) reference genome sequences
- dm3 (spike-in) reference genome sequences
- ChromInfo plain-text file listing chromosome names and sizes

## Outputs

- Combined sample+spike-in ChromInfo file with dm3 chromosomes renamed with the 'FLY' suffix
- Bowtie indexes for the concatenated hg19+dm3 synthetic genome
- Chromosome-partitioned BAM files (sample vs FLY spike-in) in map_files/

## How to apply

Download the hg19 and dm3 reference sequences and build a plain-text ChromInfo file listing human chromosomes followed by fruit fly chromosomes renamed with the 'FLY' suffix, as spikChIP requires the exogenous genome identifier (default FLY, allowed values FLY,HUMAN,MOUSE,EXO). Concatenate hg19+dm3 sequences into a synthetic genome and build Bowtie indexes against it, so mapped reads can later be split into sample and spike-in sets purely by chromosome name. Provide the ChromInfo file as the second positional argument of spikChIP and set -k for bin size (e.g. 1000 bp bins, ~3,095,665 human bins and ~120,397 fly bins). The naming convention is what allows downstream SeqCode segmentation and the sample/spike BAM split in map_files/ to remain consistent across all conditions in the configuration file.

## Related tools

- **spikChIP** (Consumes the ChromInfo file (second positional argument) and enforces the exogenous genome identifier via -e (default FLY, allowed FLY,HUMAN,MOUSE,EXO) to separate sample and spike-in bins) — https://github.com/eblancoga/spikChIP
- **Bowtie** (Indexes the concatenated hg19+dm3 synthetic genome so that fly reads align to FLY-suffixed chromosomes and can be filtered out later) — http://bowtie-bio.sourceforge.net
- **SeqCode** (Segments both genomes into non-overlapping bins (e.g. 1-kbp) using the ChromInfo file and counts reads per bin) — https://github.com/eblancoga/seqcode
- **Samtools** (Filters unmapped reads (-F 0x4) and supports splitting the filtered BAM into human and FLY spike-in read sets)
- **gawk** (Formats per-bin count tables keyed by chromosome names from the ChromInfo convention)

## Examples

```
./spikChIP -vhcde FLY -k 1 -p 0 configuration_file.txt chrominfo_file.txt
```

## Evaluation signals

- The combined ChromInfo file contains ~3,095,665 human 1-kbp bins and ~120,397 fly bins with fly chromosome names all carrying the FLY suffix
- Bowtie is run with -m 1 so multi-locus reads are discarded, and samtools -F 0x4 removes unmapped reads before the human/FLY BAM split
- Every row of the RAW per-bin output (results/FINAL_H3K79me2_25-75-75-25_RAW_1000_sample_avg.txt.gz and _max.txt.gz) is a chromosome-start-end-count tuple consistent with ChromInfo.txt
- spikChIP runs without chromosome-name errors: the configuration file's map_files/*.bam and the chrominfo_file agree on naming, and -e FLY resolves the exogenous genome

## Limitations

- The exogenous genome identifier is restricted to FLY, HUMAN, MOUSE or EXO; other exogenous species require custom naming beyond the default convention
- spikChIP by default refuses BAM files of < 1 million reads unless the -d option is used
- Fly chromosome naming must be fixed before mapping: later renaming would invalidate the Bowtie indexes and the human/spike-in split

## Evidence

- [methods] Build the combined sample+spike-in chromInfo file listing human (hg19) chromosomes and fruit fly (dm3) chromosomes renamed with the mandatory 'FLY' suffix: [evidence span withheld — anchor status: not_found]
