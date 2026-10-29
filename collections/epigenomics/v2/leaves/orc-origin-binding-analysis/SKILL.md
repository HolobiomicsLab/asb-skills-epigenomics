---
name: orc-origin-binding-analysis
description: Use when paired-end MNase-seq data exist for wild-type versus an orc1-161
  (or other ORC-mutant) yeast condition and you need RoboCOP-derived chromatin occupancy
  evaluated at ACS replication-origin sites.
when_to_use_negative:
- When the goal is general TF or nucleosome occupancy across the genome rather than
  ORC occupancy specifically at annotated replication origins (ACS sites)
- When only ATAC-seq data are available and MNase-seq performance matters — the article
  reports markedly better RoboCOP performance when using MNase-seq than ATAC-seq
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: RoboCOP
  role: Multivariate HMM that integrates MNase-seq accessibility data with nucleotide
    sequence to compute genome-wide probabilistic occupancy scores of nucleosomes
    and DNA-binding factors, from which ORC posterior probabilities at ACS origins
    are extracted
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:2376887dc446c7dca1e16bc60e38a152c80d3b470d652896971def63e9e40ea2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - RoboCOP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# orc-origin-binding-analysis

## Summary

Applies RoboCOP's multivariate HMM to paired-end MNase-seq data from wild-type and orc1-161 G2-arrested yeast to compute posterior probabilities of ORC and nucleosome occupancy at replication origins, enabling comparison of ORC binding across cell populations.

## When to use

Use when paired-end MNase-seq data exist for wild-type versus an orc1-161 (or other ORC-mutant) yeast condition and you need RoboCOP-derived chromatin occupancy evaluated at ACS replication-origin sites.

## When NOT to use

- When the goal is general TF or nucleosome occupancy across the genome rather than ORC occupancy specifically at annotated replication origins (ACS sites)
- When only ATAC-seq data are available and MNase-seq performance matters — the article reports markedly better RoboCOP performance when using MNase-seq than ATAC-seq

## Inputs

- Paired-end MNase-seq datasets for wild-type G2-arrested yeast (SRA accession SRP041314)
- Paired-end MNase-seq datasets for orc1-161 G2-arrested yeast (SRA accession SRP041314)
- Yeast nucleotide (genome) sequence for RoboCOP inference
- Annotated ACS sites in the yeast genome
- Most prevalent ORC binding motif from ORC ChIP-seq peaks
- RoboCOP configuration file (config.ini with cshared path to librobocop.so)

## Outputs

- Genome-wide probabilistic occupancy profile of nucleosomes and DNA-binding factors (posterior probabilities) for wild-type MNase-seq
- Genome-wide probabilistic occupancy profile for orc1-161 mutant MNase-seq
- ORC posterior probabilities at annotated ACS origins plus flanking nucleosome positions for both conditions
- Supplementary Figure S19 reproduction (supplementary_figure_s19.png) contrasting wild-type versus mutant ORC occupancy at origins

## How to apply

Download the paired-end MNase-seq datasets for wild-type and orc1-161 G2-arrested yeast from SRA accession SRP041314, then clone the HarteminkLab RoboCOP repository, run the gccCompile script to build librobocop.so, and register its absolute path in the configuration file via the cshared entry (following analysis/config_example.ini). Run RoboCOP's multivariate HMM on the wild-type MNase-seq data together with the nucleotide sequence to infer the genome-wide probabilistic occupancy profile of nucleosomes and DNA-binding factors, then repeat the inference on the orc1-161 mutant data to obtain the mutant occupancy profile. Extract ORC posterior probabilities at annotated ACS origins plus flanking nucleosome positions for both conditions, using the most prevalent ORC binding motif from ORC ChIP-seq peaks and annotated ACS sites in the yeast genome as the origin reference resources. Visualize the wild-type versus mutant contrast at these origins to reproduce Supplementary Figure S19. Rationale: RoboCOP computes a probabilistic occupancy landscape at single-nucleotide resolution, so origin-level ORC posterior probability can be compared directly between conditions rather than relying on peak calls.

## Related tools

- **RoboCOP** (Multivariate HMM that integrates MNase-seq accessibility data with nucleotide sequence to compute genome-wide probabilistic occupancy scores of nucleosomes and DNA-binding factors, from which ORC posterior probabilities at ACS origins are extracted) — https://github.com/HarteminkLab/RoboCOP

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git && cd RoboCOP/robocop && bash gccCompile
```

## Evaluation signals

- Reproduced figure (supplementary_figure_s19.png) shows strong ORC posterior probability at ACS origins in wild type
- Reproduced figure shows absence of ORC posterior probability at ACS origins in the orc1-161 mutant
- Flanking nucleosomes are spaced farther apart where ORC is bound in the wild-type profile
- librobocop.so builds via gccCompile and its absolute path is correctly registered under cshared in the config file
- RoboCOP inference completes on both wild-type and orc1-161 MNase-seq inputs producing genome-wide posterior probability profiles

## Limitations

- No version numbers are stated for RoboCOP or associated tools in the source context
- Accuracy of chromatin occupancy profiles improves with deeper sequencing; no sequencing depth requirements are quantified
- RoboCOP uses only MNase-seq or ATAC-seq accessibility data (MNase-seq markedly preferred), so other accessibility assays are not supported

## Evidence
