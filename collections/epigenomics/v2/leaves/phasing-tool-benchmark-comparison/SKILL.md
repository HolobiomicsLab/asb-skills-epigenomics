---
name: phasing-tool-benchmark-comparison
description: Use when a large or error-prone long-read dataset (e.g. ~32x PacBio SMRT
  for a gigantic diploid genome) must be phased and tool scalability in CPU hours
  and memory must be empirically compared.
when_to_use_negative:
- Input ploidy is higher than diploid — SpecHap's graph bisection model does not fit
  increased ploidy
- The goal is haplotype-resolved structural variation phasing — current SIH software
  including SpecHap supports only SNPs and short indels
- The input is already a phased VCF and no tool comparison or resource measurement
  is needed
edam_operation: http://edamontology.org/operation_3745
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: SpecHap
  role: Primary phasing tool under test; spectral graph-theory diploid SIH algorithm
    run in PacBio mode
  repo: https://github.com/deepomicslab/SpecHap
- name: HapCUT2
  role: Comparison phasing tool; also provides the extractHAIRS fragment-extraction
    utility used to preprocess inputs
  repo: https://github.com/vibansal/HapCUT2
- name: DCHap
  role: Comparison phasing tool (divide-and-conquer fragment partitioning + MEC optimization)
- name: ReFHap
  role: Comparison phasing tool (graph max-cut heuristic for diploid SIH)
- name: arpack-ng
  role: Dependency providing eigenvalue computation for SpecHap's spectral analysis
  repo: https://github.com/opencollab/arpack-ng
- name: htslib
  role: Dependency required for building SpecHap and reading/writing VCF/BAM
  repo: https://github.com/samtools/htslib
- name: PacBio SMRT
  role: Long-read sequencing platform supplying the ~32x axolotl benchmark data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:22e69c22aa450733d124766330a2ff25d842e14677d422f2b9d2fa800536d1c5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deepomicslab/SpecHap
  repo_url: https://github.com/deepomicslab/SpecHap
  tools:
  - DCHap
  - HapCUT2
  - PacBio SMRT
  - ReFHap
  - SpecHap
  - arpack-ng
  - htslib
  edam_operation: http://edamontology.org/operation_3745
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# phasing-tool-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks diploid haplotype-phasing tools (SpecHap, HapCUT2, DCHap, ReFHap) on the same fragment/VCF inputs, using resource monitoring to compare CPU time, peak memory, and completion under fixed usage boundaries.

## When to use

Use when a large or error-prone long-read dataset (e.g. ~32x PacBio SMRT for a gigantic diploid genome) must be phased and tool scalability in CPU hours and memory must be empirically compared.

## When NOT to use

- Input ploidy is higher than diploid — SpecHap's graph bisection model does not fit increased ploidy
- The goal is haplotype-resolved structural variation phasing — current SIH software including SpecHap supports only SNPs and short indels
- The input is already a phased VCF and no tool comparison or resource measurement is needed

## Inputs

- Position-sorted, bgzipped VCF of variant calls
- BAM alignment file (e.g. PacBio SMRT long-read alignments)
- Reference genome FASTA
- Coordinate-sorted fragment file produced by extractHAIRS

## Outputs

- Phased VCF per tool
- Resource metrics per tool (CPU time, wall time, peak resident memory)
- Recorded failure modes for tools exceeding the usage boundary

## How to apply

Extract read-fragment data from the sequencing alignments with the HapCUT2-derived extractHAIRS utility using technology-specific settings (e.g. '--pacbio 1' plus BAM, VCF, and reference arguments). Sort the fragment file in coordinate order matching the position-sorted, bgzipped VCF ('sort -n -k3' for the default PacBio fragment format) and run each phasing tool on identical inputs — for SpecHap use PacBio mode ('--vcf <gzipped VCF> --frag <sorted fragments> --out <phased VCF> --pacbio'), which builds a linkage graph and partitions alleles via spectral (Fiedler-vector) graph bisection. Wrap every invocation with a resource monitor such as '/usr/bin/time -v' to capture wall/CPU time and peak resident memory. Enforce a common usage boundary — in the reference benchmark, 5 CPU days and 40 GB memory — and record failure modes (memory overflow or time-limit exceeded) for tools that do not finish. Validate success by confirming a complete phased VCF is written within the resource envelope reported for the target dataset (e.g. ~6 CPU hours and 945 MB peak memory for Ambystoma mexicanum).

## Related tools

- **SpecHap** (Primary phasing tool under test; spectral graph-theory diploid SIH algorithm run in PacBio mode) — https://github.com/deepomicslab/SpecHap
- **HapCUT2** (Comparison phasing tool; also provides the extractHAIRS fragment-extraction utility used to preprocess inputs) — https://github.com/vibansal/HapCUT2
- **DCHap** (Comparison phasing tool (divide-and-conquer fragment partitioning + MEC optimization))
- **ReFHap** (Comparison phasing tool (graph max-cut heuristic for diploid SIH))
- **arpack-ng** (Dependency providing eigenvalue computation for SpecHap's spectral analysis) — https://github.com/opencollab/arpack-ng
- **htslib** (Dependency required for building SpecHap and reading/writing VCF/BAM) — https://github.com/samtools/htslib
- **PacBio SMRT** (Long-read sequencing platform supplying the ~32x axolotl benchmark data)

## Examples

```
/usr/bin/time -v SpecHap --vcf axolotl.vcf.gz --frag axolotl.fragments.sorted --out axolotl.phased.vcf --pacbio
```

## Evaluation signals

- A complete phased VCF is written to the --out path for the tool under test
- SpecHap's recorded CPU time falls within ~6 CPU hours and peak memory within ~945 MB for the ~32x PacBio axolotl dataset
- All comparison tools either complete within the shared boundary (5 CPU days, 40 GB memory) or have their failure mode (memory overflow vs. time limit) recorded
- Fragment file is coordinate-sorted consistently with the position-sorted bgzipped VCF (verified by the 'sort -n -k3' ordering)
- SpecHap completes all tested settings (e.g. up to 50x coverage) where other packages fail or lack data-type support

## Limitations

- No guarantee that SpecHap provides optimal solutions; the interval size parameter affects the phasing result and larger interval sizes may reduce efficiency
- The current graph bisection model does not fit increased ploidy; high-ploidy generalization is not implemented
- SpecHap supports only SNPs and short insertions/deletions; haplotype-resolved structural variation is not supported
- No software version was stated for SpecHap, complicating reproducibility

## Evidence
