---
name: n50-mismatch-rate-benchmarking
description: Use when comparing de novo assembly quality (N50, mismatch rate, and
  related Quast metrics) between a chemically treated read set (e.g. RIMS-seq alkaline-treated)
  and a standard DNA-seq control to show the treatment preserves assembly metrics.
when_to_use_negative:
- The library preparation is based on tagmentation rather than adapter ligation —
  the article states RIMS-seq does not work with tagmentation-based protocols
- An exact reference genome is available and only methylase specificity (not assembly
  quality) is needed — the RIMS-seq option 2 analysis path skips SPAdes assembly entirely
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: SPAdes
  role: Performs the de novo genome assembly of trimmed RIMS-seq and control paired-end
    reads, producing contigs FASTA for metric comparison
  repo: https://github.com/ablab/spades
- name: Quast
  role: Evaluates both SPAdes assemblies and extracts mismatch rate and N50 for the
    side-by-side comparison table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:9b2e9c285b593ce4f836722f13702fc5a9d1f18364c5a64f493f0c4f2a47ed7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/spades
  tools:
  - Quast
  - SPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# n50-mismatch-rate-benchmarking

## Summary

Benchmarks de novo bacterial genome assembly quality by comparing N50 and mismatch rate between alkaline-treated RIMS-seq reads and a DNA-seq(+3H) control, using SPAdes for assembly and Quast for metric extraction, to verify that the RIMS-seq NaOH treatment does not degrade assembly quality.

## When to use

Use when comparing de novo assembly quality (N50, mismatch rate, and related Quast metrics) between a chemically treated read set (e.g. RIMS-seq alkaline-treated) and a standard DNA-seq control to show the treatment preserves assembly metrics.

## When NOT to use

- The library preparation is based on tagmentation rather than adapter ligation — the article states RIMS-seq does not work with tagmentation-based protocols
- An exact reference genome is available and only methylase specificity (not assembly quality) is needed — the RIMS-seq option 2 analysis path skips SPAdes assembly entirely

## Inputs

- Paired-end FASTQ.gz reads from RIMS-seq (alkaline-treated) library
- Paired-end FASTQ.gz reads from DNA-seq(+3H) control library
- Trimmed (adapter/quality-filtered) FASTQ files for both libraries

## Outputs

- SPAdes contigs FASTA for the RIMS-seq assembly
- SPAdes contigs FASTA for the DNA-seq control assembly
- Quast report with N50 and mismatch rate for each assembly
- Side-by-side comparison table (quast_assembly_comparison.tsv)

## How to apply

Download the paired-end RIMS-seq (alkaline-treated) and DNA-seq(+3H) control reads for the same strain (e.g. Acinetobacter calcoaceticus ATCC 49823 from BioProject PRJNA706563) and apply identical adapter/quality trimming to both. Assemble each trimmed read set de novo with SPAdes to produce contigs FASTA files; per the RIMS-seq pipeline, the genome assembly step with SPAdes is used when no exact reference genome is available (option 1). Evaluate both assemblies with Quast and extract the mismatch rate and N50 for each. Tabulate the RIMS-seq vs control metrics side by side (e.g. quast_assembly_comparison.tsv). The benchmark passes if the RIMS-seq assembly's mismatch rate and N50 are comparable to the DNA-seq control, consistent with the finding that RIMS-seq sequencing quality is identical to DNA-seq and permits genome assembly at metrics comparable to standard shotgun sequencing.

## Related tools

- **SPAdes** (Performs the de novo genome assembly of trimmed RIMS-seq and control paired-end reads, producing contigs FASTA for metric comparison) — https://github.com/ablab/spades
- **Quast** (Evaluates both SPAdes assemblies and extracts mismatch rate and N50 for the side-by-side comparison table)

## Examples

```
spades.py --pe1-1 rims_R1.trim.fastq.gz --pe1-2 rims_R2.trim.fastq.gz -o rims_spades; spades.py --pe1-1 ctrl_R1.trim.fastq.gz --pe1-2 ctrl_R2.trim.fastq.gz -o ctrl_spades; quast.py rims_spades/contigs.fasta ctrl_spades/contigs.fasta -o quast_cmp
```

## Evaluation signals

- The RIMS-seq assembly's mismatch rate is comparable to the DNA-seq(+3H) control assembly (consistent with Figure 3B / Supplementary Table S1)
- The RIMS-seq assembly's N50 is comparable to the control's N50
- Both read sets received identical adapter and quality trimming before assembly, so metric differences reflect treatment not preprocessing
- Quast reports for both assemblies exist and the metrics are tabulated side by side in quast_assembly_comparison.tsv

## Limitations

- The RIMS-seq pipeline README notes the genome assembly step (SPAdes) is optional and only recommended when an exact reference genome is unavailable; with option 2 the reference must be exactly the same strain
- Assembly metric comparability was demonstrated for the A. calcoaceticus ATCC 49823 case (Figure 3B, Supplementary Table S1); other organisms or treatments may need re-validation
- RIMS-seq library preparation must be adapter-ligation based; tagmentation-based protocols are incompatible, so downstream assembly benchmarking inherits that constraint

## Evidence

- [intro] assembly at metrics comparable to shotgun: "RIMS-seq de novo identifies novel activities without the need for a reference genome and permits the assembly of the bacterial genome at metrics comparable to standard shotgun sequencing."
- [methods] SPAdes assembly step in pipeline: "It includes a genome assembly step using SPAdes and uses this assembled genome for the downstream analysis."
- [readme] SPAdes optional, option 1 vs 2: "if you have a reference genome and perform you own mapping, SPAdes and BWA are not necessary (see option 2)."
- [abstract] sequencing quality identical to DNA-seq: "the resulting sequencing quality is identical to DNA-seq, enabling RIMS-seq to substitute standard sequencing of bacterial genomes"
