---
name: transcript-level-estimate-reconciliation-across-sequencing-technologies
description: Use when two RNA-Seq libraries from different sequencing platforms (e.g.
  SOLiD MAQC WT-protocol and Illumina SRA012427) need expression estimates compared,
  and you must quantify how fragment-bias correction improves cross-platform agreement.
when_to_use_negative:
- Libraries are generated on the same sequencing platform with the same sample preparation
  — bias correction regimes are designed to explain cross-technology discordance,
  not within-platform variation.
- 'Input is raw reads without alignments: eXpress-D requires fragment alignments plus
  reference transcript sequences, not FASTQ alone.'
- You only need within-library differential expression ranking; this skill targets
  cross-platform correlation of absolute expression estimates.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: eXpress-D
  role: Runs the distributed likelihood-based ambiguous fragment assignment and expression
    estimation under each of the four bias-correction regimes
  repo: https://github.com/adarob/express-d/wiki
- name: Python
  role: Joins transcript-level estimates by identifier across platforms and computes
    Pearson R-squared per regime (e.g. scipy.stats or numpy)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:e62a43dd32ed1ed4105644fa89ee657f57956a9cfec7425e0e397180a32f89a6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - eXpress-D
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# transcript-level-estimate-reconciliation-across-sequencing-technologies

## Summary

Reconciles transcript-level RNA-Seq expression estimates generated on different sequencing technologies (Illumina vs SOLiD) by running likelihood-based estimation in eXpress-D under four fragment-bias-correction regimes and joining estimates by transcript identifier to compute cross-platform Pearson R-squared.

## When to use

Use when two RNA-Seq libraries from different sequencing platforms (e.g. SOLiD MAQC WT-protocol and Illumina SRA012427) need expression estimates compared, and you must quantify how fragment-bias correction improves cross-platform agreement.

## When NOT to use

- Libraries are generated on the same sequencing platform with the same sample preparation — bias correction regimes are designed to explain cross-technology discordance, not within-platform variation.
- Input is raw reads without alignments: eXpress-D requires fragment alignments plus reference transcript sequences, not FASTQ alone.
- You only need within-library differential expression ranking; this skill targets cross-platform correlation of absolute expression estimates.

## Inputs

- SOLiD WT-protocol MAQC alignment files (produced with BioScope)
- Illumina SRA012427 alignment files
- Reference transcript sequences (FASTA)
- Transcript identifier mapping for joining estimates across platforms

## Outputs

- cross_platform_r2.csv with one row per correction regime (none, Illumina-only, SOLiD-only, both) and its Pearson R2
- Transcript-level expression estimate tables per library per bias-correction regime

## How to apply

Load the SOLiD WT-protocol MAQC alignments (produced with BioScope) and the Illumina SRA012427 alignments together with the reference transcript sequences into the eXpress-D execution environment. Run likelihood-based expression estimation four times per library: bias correction disabled on both platforms, enabled only on the Illumina library, enabled only on the SOLiD library, and enabled on both — the four regimes isolate each platform's contribution to cross-platform discordance. Join the resulting transcript-level expression estimates from the two platforms by transcript identifier within each correction regime. Compute Pearson R-squared between SOLiD and Illumina estimates per regime using Python (scipy.stats or numpy). Expected R2 values are 0.74 uncorrected, 0.88 with Illumina-only correction, 0.85 with SOLiD-only correction, and 0.94 when both platforms are corrected, demonstrating that bias correction improves replicability across sequencing technologies.

## Related tools

- **eXpress-D** (Runs the distributed likelihood-based ambiguous fragment assignment and expression estimation under each of the four bias-correction regimes) — https://github.com/adarob/express-d/wiki
- **Python** (Joins transcript-level estimates by identifier across platforms and computes Pearson R-squared per regime (e.g. scipy.stats or numpy))

## Examples

```
from scipy.stats import pearsonr; r2 = pearsonr(solid_expr, illumina_expr)[0]**2
```

## Evaluation signals

- cross_platform_r2.csv exists and contains all four correction regimes (none, Illumina-only, SOLiD-only, both).
- Reproduced R2 values match 0.74, 0.88, 0.85, and 0.94 within +/-0.02.
- Uncorrected R2 (0.74) is the lowest and both-corrected R2 (0.94) is the highest — the monotonic improvement pattern must hold.
- Joined estimate tables have transcript identifiers matched 1:1 between SOLiD and Illumina within each regime, with no unmatched identifiers dropped silently.

## Limitations

- Requires alignments and reference transcript sequences compatible with eXpress-D's mseq input; standard SAM/FASTA must be converted.
- The bias-correction model assumes non-uniform fragment distribution arises from library-preparation biochemistry; protocols outside the tested variety (single-/paired-end, strand-specific/non-specific) may not benefit.
- Results are demonstrated on the MAQC/SRA012427 benchmark; exact R2 gains may differ for other tissue or protocol combinations.

## Evidence

- [results] Cross-platform correlation between SOLiD and Illumina expression estimates increased from R2 = 0.74 uncorrected to 0.88 with Illumina-only correction: [evidence span withheld — anchor status: not_found]
- [abstract] correction of bias leads to improved replicability of results across libraries and sequencing technologies: "correction of bias leads to improved replicability of results across libraries and sequencing technologies"
- [intro] One of our main findings is that bias correction improves the correlation of expression estimates obtained from sequence data generated using different sample preparations and different sequencing: "One of our main findings is that bias correction improves the correlation of expression estimates obtained from sequence data generated using different sample preparations and different sequencing"
- [supplementary] Python script that converts standard SAM and FASTA input into the mseq input format.: "Python script that converts standard SAM and FASTA input into the mseq input format."
- [readme] Distributed ambiguous fragment assignment using [eXpress](http://bio.math.berkeley.edu/eXpress) and [Spark](http://spark.incubator.apache.org/).: "Distributed ambiguous fragment assignment using [eXpress](http://bio.math.berkeley.edu/eXpress) and [Spark](http://spark.incubator.apache.org/)."
