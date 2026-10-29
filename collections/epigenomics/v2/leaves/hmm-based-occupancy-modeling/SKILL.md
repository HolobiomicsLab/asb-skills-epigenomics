---
name: hmm-based-occupancy-modeling
description: Use when you have paired-end MNase-seq and/or ATAC-seq data plus genome
  sequence and need probabilistic genome-wide occupancy profiles of nucleosomes and
  DNA-binding factors, especially to benchmark against NucleoATAC, DANPOS2, or MACS2+FIMO.
when_to_use_negative:
- You only need a small set of TF binding sites from peak calling — a MACS2 peak/footprint
  identification followed by FIMO TF-labeling pipeline is the alternative the paper
  benchmarks against, though RoboCOP outperforms it
- Your data are single-end or not chromatin accessibility data (MNase-seq/ATAC-seq)
  — RoboCOP currently uses only MNase-seq or ATAC-seq as input
- You need occupancy profiles for organisms/data types outside the model's current
  scope (yeast, paired-end MNase-seq/ATAC-seq) without adapting the model
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: RoboCOP
  role: Multivariate state space (HMM) model that integrates nucleotide sequence with
    chromatin accessibility data to jointly compute probabilistic nucleosome and DBF
    occupancy scores
  repo: https://github.com/HarteminkLab/RoboCOP
- name: NucleoATAC
  role: Baseline nucleosome-position caller run on the same ATAC-seq data for AUPR
    comparison (RoboCOP_ATAC positions many more nucleosomes)
- name: DANPOS2
  role: Baseline nucleosome-position caller run on the same MNase-seq data for AUPR
    comparison against RoboCOP(MNase+ATAC)
- name: MACS2
  role: Alternative peak/footprint identification approach (followed by FIMO labeling)
    used as a comparison baseline
- name: GCC
  role: Used via 'bash gccCompile' to generate the shared library librobocop.so that
    the RoboCOP engine invokes
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:ba094a006a518bfbf094b49c27ef6cb0060877db33d203e166a2691e138672d8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - DANPOS2
  - GCC
  - MACS2
  - NucleoATAC
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hmm-based-occupancy-modeling

## Summary

Apply RoboCOP, a multivariate state space/hidden Markov model that integrates chromatin accessibility data (MNase-seq and/or ATAC-seq) with nucleotide sequence, to jointly compute genome-wide probabilistic occupancy scores for nucleosomes and hundreds of DNA-binding factors at single-nucleotide resolution.

## When to use

Use when you have paired-end MNase-seq and/or ATAC-seq data plus genome sequence and need probabilistic genome-wide occupancy profiles of nucleosomes and DNA-binding factors, especially to benchmark against NucleoATAC, DANPOS2, or MACS2+FIMO.

## When NOT to use

- You only need a small set of TF binding sites from peak calling — a MACS2 peak/footprint identification followed by FIMO TF-labeling pipeline is the alternative the paper benchmarks against, though RoboCOP outperforms it
- Your data are single-end or not chromatin accessibility data (MNase-seq/ATAC-seq) — RoboCOP currently uses only MNase-seq or ATAC-seq as input
- You need occupancy profiles for organisms/data types outside the model's current scope (yeast, paired-end MNase-seq/ATAC-seq) without adapting the model

## Inputs

- Paired-end MNase-seq data (GEO accession GSE153609)
- ATAC-seq data (GEO accession GSE66386)
- Nucleotide sequence of the reference genome
- Configuration file (config.ini) with cshared path to librobocop.so

## Outputs

- Genome-wide probabilistic occupancy profile of nucleosomes at single-nucleotide resolution
- Genome-wide probabilistic occupancy scores for ~150 transcription factors
- Nucleosome-position calls for benchmarking (e.g. AUPR scores against chemical cleavage annotations)

## How to apply

Clone the RoboCOP repository (github.com/HarteminkLab/RoboCOP), create the conda environment from robocop-spec-file.txt, and run 'bash gccCompile' inside the robocop directory to generate the shared library librobocop.so. Set the absolute path to librobocop.so in the config.ini file under the cshared entry (e.g. cshared=/home/myhome/RoboCOP/pkg/robocop/librobocop.so), following the example in analysis/config_example.ini. Run RoboCOP with ATAC-seq alone (GSE66386), MNase-seq alone (GSE153609), or MNase-seq combined with ATAC-seq to compute probabilistic nucleosome and TF occupancy profiles across the yeast genome; the introductory vignette is analysis/example_robocop.ipynb. Validate nucleosome positioning predictions against high-precision chemical cleavage method annotations, and TF binding predictions against ChIP-chip, ChIP-exo, and ORGANIC annotations, e.g. by computing nucleosome-position AUPR against baselines such as NucleoATAC and DANPOS2. Rationale: MNase-seq typically yields markedly better performance than ATAC-seq because MNase exonuclease activity reflects DBF sizes, and RoboCOP run on ATAC-seq alone still positions more nucleosomes than NucleoATAC on the same data.

## Related tools

- **RoboCOP** (Multivariate state space (HMM) model that integrates nucleotide sequence with chromatin accessibility data to jointly compute probabilistic nucleosome and DBF occupancy scores) — https://github.com/HarteminkLab/RoboCOP
- **NucleoATAC** (Baseline nucleosome-position caller run on the same ATAC-seq data for AUPR comparison (RoboCOP_ATAC positions many more nucleosomes))
- **DANPOS2** (Baseline nucleosome-position caller run on the same MNase-seq data for AUPR comparison against RoboCOP(MNase+ATAC))
- **MACS2** (Alternative peak/footprint identification approach (followed by FIMO labeling) used as a comparison baseline)
- **GCC** (Used via 'bash gccCompile' to generate the shared library librobocop.so that the RoboCOP engine invokes) — https://github.com/HarteminkLab/RoboCOP

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git && cd RoboCOP/robocop && bash gccCompile
```

## Evaluation signals

- All compared methods' nucleosome-position AUPR values are computed on the identical validation set (chemical cleavage annotations)
- RoboCOP_ATAC positions more nucleosomes than NucleoATAC on the same ATAC-seq data (Supplementary Figure S12A)
- RoboCOP(MNase+ATAC) AUPR exceeds DANPOS2 on the same MNase-seq data
- The rendered supplementary_figure_S12_aupr_comparison.png reproduces the Supplementary Figure S12 layout
- librobocop.so exists at the absolute path given in config.ini's cshared entry and RoboCOP runs without library-load errors

## Limitations

- MNase-seq gives markedly better RoboCOP performance than ATAC-seq, so ATAC-only profiles are less accurate (for two primary reasons, including fragment-distribution and MNase exonuclease activity)
- No version numbers stated for RoboCOP, NucleoATAC, MACS2, or FIMO; sequencing depth requirements are not quantified (accuracy improves with deeper sequencing)
- Prior information about DBF transition probabilities is not incorporated (deferred to future work)

## Evidence
