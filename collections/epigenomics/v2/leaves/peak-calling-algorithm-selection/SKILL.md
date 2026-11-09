---
name: peak-calling-algorithm-selection
description: Use when building or editing a NEAT project's Targets.txt for ChIPseq
  data (e.g. H3K4me3 ChIPseq from mouse embryonic stem cells) and you must choose
  between MACS and SPP as the peak caller.
when_to_use_negative:
- The project is RNAseq (differential gene expression) rather than ChIPseq — peak
  calling modules do not apply
- The pipeline is not managed by NEAT's Targets.txt file (e.g. a Galaxy or systemPipeR
  workflow where peak callers are configured differently)
- Reads have not yet been mapped and filtered (map and filter modules must precede
  peakcalling)
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3678
tools:
- name: NEAT
  role: Framework whose centralized Targets.txt file encodes the peak-calling algorithm
    selection and dispatches the 'peakcalling' module by exact word matching on the
    Steps_to_execute_pipe line
  repo: https://github.com/pschorderet/NEAT
- name: systemPipeR
  role: R/Bioconductor package used by NEAT's QC module in the pipeline that precedes
    peak calling
  repo: https://github.com/tgirke/systemPipeR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:5e21851767662f09fcb3f92a0d4085bc6b65269e30ebf1370f2976329df91816
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - NEAT
  - systemPipeR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# peak-calling-algorithm-selection

## Summary

Select and configure the peak-calling algorithm (MACS or SPP) for a NEAT ChIPseq pipeline by declaring the 'peakcalling' module in the centralized Targets.txt project file, alongside the alignment (bwa/bowtie) and filtering parameters it depends on.

## When to use

Use when building or editing a NEAT project's Targets.txt for ChIPseq data (e.g. H3K4me3 ChIPseq from mouse embryonic stem cells) and you must choose between MACS and SPP as the peak caller.

## When NOT to use

- The project is RNAseq (differential gene expression) rather than ChIPseq — peak calling modules do not apply
- The pipeline is not managed by NEAT's Targets.txt file (e.g. a Galaxy or systemPipeR workflow where peak callers are configured differently)
- Reads have not yet been mapped and filtered (map and filter modules must precede peakcalling)

## Inputs

- Targets.txt project file (NEAT centralized project file) copied into the new project directory
- Compressed fastq files listed under 'OriFileName' with user-defined sample names, replicate/sample-to-input relationships, and antibody specificity (e.g. H3K4me3)
- AdvancedSettings.txt containing filtering parameters (duplicate reads, min/max fragment size)
- Mapped/filter reads from the upstream 'map' and 'filter' modules (bam/bai files)

## Outputs

- Called peaks for each ChIPseq sample produced by the 'peakcalling' module using MACS or SPP
- Updated Targets.txt with the peak-calling algorithm declared on the 'Steps_to_execute_pipe' line
- Downstream GRanges objects (via the 'granges' module) for downstream analysis

## How to apply

Open the project's Targets.txt file (copied from the template shipped with the NEAT package) and set the peak-calling algorithm to either MACS or SPP; this choice must be consistent with the mapping algorithm (bwa or bowtie) and the filtering parameters defined in AdvancedSettings.txt (duplicate-read handling, minimum and maximum fragment size). If a spike-in control is used, fill the ChIP-RX (chiprx) normalization parameters; otherwise set all ChIP-RX parameters to '-'. Add the plain-word block name 'peakcalling' to the 'Steps_to_execute_pipe' line using exact word matching, since NEAT dispatches modules by literal string match, and delete or rename any block that should not run. Validation: Targets.txt must contain the alignment, filtering and peak-calling parameters plus cluster settings, and the block name must exactly match NEAT's plain-word module names.

## Related tools

- **NEAT** (Framework whose centralized Targets.txt file encodes the peak-calling algorithm selection and dispatches the 'peakcalling' module by exact word matching on the Steps_to_execute_pipe line) — https://github.com/pschorderet/NEAT
- **systemPipeR** (R/Bioconductor package used by NEAT's QC module in the pipeline that precedes peak calling) — https://github.com/tgirke/systemPipeR

## Examples

```
Steps_to_execute_pipe = unzip, QC, chiprx, map, filter, peakcalling, cleanfiles, granges
```

## Evaluation signals

- Targets.txt exists in the project directory and contains all required fields including alignment, filtering and peak-calling parameters plus cluster paths
- The 'Steps_to_execute_pipe' line contains the exact plain-word block 'peakcalling' (NEAT uses exact word matching); no renamed/deleted blocks (e.g. 'chiprs_NO') remain on the line
- ChIP-RX parameters are either fully populated (spike-in control used) or all set to '-' (no spike-in), not mixed
- The selected peak caller is one of NEAT's supported algorithms (MACS or SPP) and is consistent with the chosen mapping algorithm (bwa or bowtie) and AdvancedSettings.txt filtering parameters
- The pipeline completes the peakcalling step downstream of 'unzip', 'QC', 'map' and 'filter' blocks without module-name mismatch errors

## Limitations

- NEAT only offers MACS and SPP as peak-calling algorithms; other peak callers are not supported through Targets.txt
- NEAT uses exact word matching on module names, so a typo or renaming (e.g. 'chiprs_NO') silently changes which blocks run
- If no spike-in control is used, all ChIP-RX parameters must be set to '-' or the chiprx module will misbehave
- NEAT is under development (devel mode per README) and limited to Mac OSx; no changelog is available in the repository

## Evidence

- [methods] The ‘peakcalling’ module specifies the algorithm used to call peaks.: "The ‘peakcalling’ module specifies the algorithm used to call peaks."
- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters as well as: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters as well as"
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [other] Populate the 'Steps_to_execute_pipe' line with exact plain-word building blocks, deleting or renaming any block that should not run: [evidence span withheld — anchor status: not_found]
