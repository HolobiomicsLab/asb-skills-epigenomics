---
name: wildtype-versus-mutant-comparative-visualization
description: Use when paired-end MNase-seq data exist for two matched cell populations
  (e.g. wild type and a temperature-sensitive mutant such as orc1-161 under G2 arrest)
  and you need to visualize condition-specific DBF/nucleosome occupancy at annotated
  loci such as replication origins.
when_to_use_negative:
- Input data are ATAC-seq rather than MNase-seq — the article reports markedly better
  RoboCOP performance with MNase-seq because MNase exonuclease activity reflects DBF
  sizes
- The goal is peak/footprint calling rather than probabilistic occupancy inference
  — MACS2 followed by FIMO TF-labeling is the alternative approach and is explicitly
  outperformed by RoboCOP
- Only one condition is available and no wild-type/mutant contrast can be formed
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RoboCOP
  role: Multivariate HMM run on wild-type and orc1-161 MNase-seq data with nucleotide
    sequence to infer condition-specific occupancy profiles of nucleosomes and DNA-binding
    factors (including ORC posterior probabilities)
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:ce17aaf125ca393826a56a3ace8b6b920f6f702e9423384c4e166b861a1daa23
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - RoboCOP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# wildtype-versus-mutant-comparative-visualization

## Summary

Contrasts RoboCOP-derived chromatin occupancy (ORC posterior probabilities and flanking nucleosome positions) between wild-type and orc1-161 mutant G2-arrested yeast to reproduce the paper's Supplementary Figure S19, using annotated ACS origins and the most prevalent ORC binding motif from ORC ChIP-seq peaks as reference.

## When to use

Use when paired-end MNase-seq data exist for two matched cell populations (e.g. wild type and a temperature-sensitive mutant such as orc1-161 under G2 arrest) and you need to visualize condition-specific DBF/nucleosome occupancy at annotated loci such as replication origins.

## When NOT to use

- Input data are ATAC-seq rather than MNase-seq — the article reports markedly better RoboCOP performance with MNase-seq because MNase exonuclease activity reflects DBF sizes
- The goal is peak/footprint calling rather than probabilistic occupancy inference — MACS2 followed by FIMO TF-labeling is the alternative approach and is explicitly outperformed by RoboCOP
- Only one condition is available and no wild-type/mutant contrast can be formed

## Inputs

- Paired-end MNase-seq datasets for wild-type G2-arrested yeast (SRA SRP041314)
- Paired-end MNase-seq datasets for orc1-161 mutant G2-arrested yeast (SRA SRP041314)
- Nucleotide sequence of the yeast genome
- Annotated ACS replication-origin sites in the yeast genome
- Most prevalent ORC binding motif derived from ORC ChIP-seq peaks

## Outputs

- Genome-wide probabilistic occupancy profile of nucleosomes and DNA-binding factors for the wild-type condition
- Genome-wide probabilistic occupancy profile for the orc1-161 mutant condition
- ORC posterior probabilities at ACS origins plus flanking nucleosome positions for both conditions
- Comparative figure (supplementary_figure_s19.png) reproducing Supplementary Figure S19

## How to apply

Download the paired-end MNase-seq datasets for both conditions from SRA accession SRP041314, then clone the HarteminkLab RoboCOP repository and build librobocop.so with the gccCompile script, registering its absolute path as the cshared entry in the configuration file (following analysis/config_example.ini). Run RoboCOP's multivariate HMM separately on the wild-type and orc1-161 MNase-seq data together with nucleotide sequence to infer genome-wide probabilistic occupancy of nucleosomes and DNA-binding factors in each condition. As reference loci, use the most prevalent ORC binding motif from ORC ChIP-seq peaks together with annotated ACS sites in the yeast genome. Extract ORC posterior probabilities at annotated ACS origins plus flanking nucleosome positions for both conditions and plot the wild-type versus mutant contrast, saving the output as supplementary_figure_s19.png. Rationale: RoboCOP integrates chromatin accessibility data with nucleotide sequence via a multivariate HMM, and MNase-seq gives markedly better RoboCOP performance than ATAC-seq because fragment distributions reflect DBF sizes.

## Related tools

- **RoboCOP** (Multivariate HMM run on wild-type and orc1-161 MNase-seq data with nucleotide sequence to infer condition-specific occupancy profiles of nucleosomes and DNA-binding factors (including ORC posterior probabilities)) — https://github.com/HarteminkLab/RoboCOP

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git; cd RoboCOP/robocop; bash gccCompile
```

## Evaluation signals

- The reproduced figure shows strong ORC posterior probability at ACS origins in the wild-type profile
- The reproduced figure shows absence of ORC posterior probability at ACS origins in the orc1-161 mutant profile
- Flanking nucleosomes are spaced farther apart at origins where ORC is bound
- Both occupancy profiles were inferred from the same MNase-seq accession (SRP041314) and identical genome sequence inputs
- The output is saved as supplementary_figure_s19.png and visually matches the paper's Supplementary Figure S19 contrast

## Limitations

- No version numbers are stated for RoboCOP (or comparison tools such as MACS2, FIMO, NucleoATAC), so exact reproducibility depends on the current repository state
- Accuracy of the chromatin occupancy profiles improves with deeper sequencing; sequencing depth requirements are not quantified
- MNase-seq gives markedly better performance than ATAC-seq, so results are not transferable to ATAC-seq inputs without loss of accuracy
- Inference of prior information about DBF transition probabilities was deferred to future work, so TF-specific dynamics may be imperfectly captured

## Evidence

- [other] Expected visual contrast between conditions: [evidence span withheld — anchor status: not_found]
