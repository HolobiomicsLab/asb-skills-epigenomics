---
name: hichip-plac-seq-pseudolinker-handling
description: Use when processing paired-end HiChIP or PLAC-seq FASTQ data whose junction
  sequence between the two genomic tags differs from the ChIA-PET bridge linker and
  needs assay-aware linker scanning before mapping.
when_to_use_negative:
- Input data has already been linker-filtered and partitioned (e.g., only full-PET
  reads remain) — proceed directly to mapping instead
- Data type lacks a junction sequence between two genomic tags (e.g., non-ligation
  assays) — the linker-scan categorization logic does not apply
edam_operation: http://edamontology.org/operation_3695
tools:
- name: ChIA-PIPE
  role: Automated pipeline whose linker-filtering step scans read pairs and categorizes
    them by bridge-linker/junction detection; invoked via 0.chia_pipe_shell.sh with
    a config file
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: ChIA-PET Utilities (CPU)
  role: Provides the CPU linker_filter module that scans every read for the bridge
    linker sequence between the two genomic tags
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:e347f3b76f7a6422a63eacd87f37e8b2f18a29c8c32160dd29557a629d62d936
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - ChIA-PET Utilities (CPU)
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# hichip-plac-seq-pseudolinker-handling

## Summary

Configure ChIA-PIPE's linker-filtering stage (via the CPU linker_filter module) so that paired-end reads from related chromatin-mapping assays whose junction sequence between the two genomic tags differs from the classic ChIA-PET bridge linker — e.g., HiChIP and PLAC-seq — are correctly scanned and partitioned.

## When to use

Use when processing paired-end HiChIP or PLAC-seq FASTQ data whose junction sequence between the two genomic tags differs from the ChIA-PET bridge linker and needs assay-aware linker scanning before mapping.

## When NOT to use

- Input data has already been linker-filtered and partitioned (e.g., only full-PET reads remain) — proceed directly to mapping instead
- Data type lacks a junction sequence between two genomic tags (e.g., non-ligation assays) — the linker-scan categorization logic does not apply

## Inputs

- Paired-end ChIA-PET/HiChIP/PLAC-seq reads in FASTQ format (e.g., LDK0004-ds_*.fastq.gz test data from Dropbox)
- ChIA-PIPE configuration file specifying the assay's junction/linker sequence and read mode (short-read 2 × 21 bp or long-read 2 × 150 bp)
- Reference genome for downstream tag mapping (BWA-ALN/BWA-MEM hybrid)

## Outputs

- Partitioned read-pair files for the three categories: (i) no linker, (ii) linker with one usable tag, (iii) full PET with genuine linker
- linker_filter_stats.txt tallying the counts of each category
- Full-PET (category iii) read set passed forward to mapping and loop detection

## How to apply

Place the paired-end FASTQ files in a fastq/ directory and set the assay's junction/linker sequence in a ChIA-PIPE configuration file, since an effective pipeline must handle ChIA-PET, HiChIP, and PLAC-seq 'despite different junction sequences between the two tags.' Launch chia_pipe-master/0.chia_pipe_shell.sh with the config so the CPU linker_filter module scans every read pair for the expected junction sequence flanked by the two genomic tags, and partition read pairs into (i) no linker detected, (ii) a linker and one usable genomic tag, or (iii) a linker and full PETs. The pipeline is configurable for either short-read (2 × 21 bp) or long-read (2 × 150 bp) data, so the tag lengths in the config must match the assay's sequencing mode. Only the full-PET category (iii) read pairs are carried forward to loop detection; the flanking tag sequences are subsequently mapped to the reference genome using a hybrid of BWA-ALN and BWA-MEM.

## Related tools

- **ChIA-PIPE** (Automated pipeline whose linker-filtering step scans read pairs and categorizes them by bridge-linker/junction detection; invoked via 0.chia_pipe_shell.sh with a config file) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **ChIA-PET Utilities (CPU)** (Provides the CPU linker_filter module that scans every read for the bridge linker sequence between the two genomic tags) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- linker_filter_stats.txt reports all three categories with counts that partition the total read pairs
- Only the full-PET (category iii) read pairs are flagged/carried forward to loop detection, per the pipeline's rule that 'Only PET reads in (iii) are used for detection of interaction loops'
- The config's junction sequence and tag-length setting (2 × 21 bp vs 2 × 150 bp) match the assay's sequencing mode before launch
- Downstream tag alignment succeeds using the BWA-ALN/BWA-MEM hybrid against the target reference genome

## Limitations

- The reference genome version used for alignment is not specified in the provided article section
- Only PET reads with a detected linker and both genomic tags (category iii) enter loop detection; categories (i) and (ii) are dropped, so poor linker detection inflates the discarded fraction
- The junction sequence differs between ChIA-PET, HiChIP, and PLAC-seq, so the config must be set per assay — a mismatch causes most read pairs to fall into the 'no linker' category

## Evidence

- [readme] CPU collection purpose: [evidence span withheld — anchor status: not_found]
