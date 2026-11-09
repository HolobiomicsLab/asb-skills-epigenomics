---
name: lncrna-transcript-sequence-retrieval
description: Use when a study needs canonical lncRNA transcript sequences (e.g. the
  7 TPX-validated lncRNAs MEG3, Meg3, lncSmad7, NEAT1, TERC, CDKN2B-AS1) from a fixed
  annotation release before running sequence-based prediction tools.
when_to_use_negative:
- The analysis targets coding-gene promoters or DNA regions only and no ssRNA sequence
  is needed (3plex dsDNA.fa input comes from genomic intervals, not lncRNA transcripts)
- A published lncRNA isoform has been experimentally characterized and a specific
  non-longest transcript should be used instead of the longest-transcript heuristic
- The annotation release must differ from the rest of the pipeline (mixing GENCODE
  v32 coordinates with other builds breaks bedtools-shuffled negative matching)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0099
tools:
- name: GENCODE
  role: Source annotation (version 32, M25 gene symbols) from which the longest transcript
    sequences of the 7 TPX-validated lncRNAs are extracted
- name: 3plex
  role: Downstream consumer of the retrieved FASTA sequences as the ssRNA.fa single-sequence
    input for TPX prediction
  repo: https://github.com/molinerisLab/3plex
- name: RNAplfold
  role: Computes per-nucleotide unpaired probabilities on the retrieved transcripts
    to build ss10/ss20/ss50 masked variants
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:3da3d68e8abbcf6eb9d2889a63abbb89d51a590458a3e465af750c5ee6c26896
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/molinerisLab/3plex
  repo_url: https://github.com/molinerisLab/3plex
  tools:
  - 3plex
  - GENCODE
  - RNAplfold
  - ViennaRNA
  - bedtools
  - pROC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lncRNA-transcript-sequence-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves the longest transcript FASTA sequences for target lncRNAs from a genome annotation release (GENCODE v32) as the ssRNA input required for downstream triplex (TPX) prediction with 3plex and RNAplfold-based secondary-structure masking.

## When to use

Use when a study needs canonical lncRNA transcript sequences (e.g. the 7 TPX-validated lncRNAs MEG3, Meg3, lncSmad7, NEAT1, TERC, CDKN2B-AS1) from a fixed annotation release before running sequence-based prediction tools.

## When NOT to use

- The analysis targets coding-gene promoters or DNA regions only and no ssRNA sequence is needed (3plex dsDNA.fa input comes from genomic intervals, not lncRNA transcripts)
- A published lncRNA isoform has been experimentally characterized and a specific non-longest transcript should be used instead of the longest-transcript heuristic
- The annotation release must differ from the rest of the pipeline (mixing GENCODE v32 coordinates with other builds breaks bedtools-shuffled negative matching)

## Inputs

- List of lncRNA gene symbols (e.g. MEG3, lncSmad7, NEAT1, TERC, CDKN2B-AS1)
- GENCODE version 32 annotation (GTF plus transcript FASTA)
- Genome build identifier consistent with the annotation release (GRCh38/M25 gene symbols)

## Outputs

- FASTA file with the longest transcript sequence per lncRNA (one sequence per 3plex ssRNA.fa run)
- Set of unmasked and masked sequence variants (ss0, ss10, ss20, ss50) for secondary-structure filtering

## How to apply

Query the chosen genome annotation release (the paper used GENCODE version 32 with M25 gene symbols) for the genes of interest and extract the longest transcript per gene to serve as the representative ssRNA sequence. Emit each transcript as a single-entry multi-FASTA (3plex requires the ssRNA.fa file to contain only one sequence per run). Use these sequences as input to RNAplfold (ViennaRNA) to compute per-nucleotide unpaired probabilities and to generate masked variants (ss0/ss10/ss20/ss50) before scoring with 3plex. Fixing one annotation version and one transcript-per-gene rule keeps the AUC comparisons across masking levels and lncRNAs reproducible.

## Related tools

- **GENCODE** (Source annotation (version 32, M25 gene symbols) from which the longest transcript sequences of the 7 TPX-validated lncRNAs are extracted)
- **3plex** (Downstream consumer of the retrieved FASTA sequences as the ssRNA.fa single-sequence input for TPX prediction) — https://github.com/molinerisLab/3plex
- **RNAplfold** (Computes per-nucleotide unpaired probabilities on the retrieved transcripts to build ss10/ss20/ss50 masked variants)

## Examples

```
docker run -u `id -u`:`id -g` -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/MEG3_longest.fa $PWD/dsDNA.fa $PWD/results_3plex
```

## Evaluation signals

- Each retrieved FASTA entry matches the longest transcript of the corresponding gene in GENCODE v32 and carries consistent gene symbols (M25)
- Exactly 7 TPX-validated lncRNA sequences are present, matching the 7 x 4 (lncRNA x masking level) design expected in ss_masking_auc_comparison.csv
- Each ssRNA.fa file fed to 3plex contains only one sequence, as required by the 3plex input specification
- Sequence coordinates/identifiers remain consistent with the genome build used for the bedtools-shuffled genomic negatives and ENCODE blacklist filtering

## Limitations

- Only 2 out of 7 lncRNAs had IDR conservative peaks, so transcript choice interacts with downstream peak-set availability; annotation release (v32) fixes the transcript universe and results may shift with other releases
- The longest-transcript rule is a heuristic; 3plex additionally restricts searches to single-stranded regions predicted by RNAplfold, so transcript-level sequence choice affects masking outcomes

## Evidence

- [other] Retrieve the longest transcript sequences of the 7 TPX-validated lncRNAs from GENCODE v32.: [evidence span withheld — anchor status: not_found]
- [readme] 3plex ssRNA.fa must contain only one sequence: [evidence span withheld — anchor status: external_doc]
