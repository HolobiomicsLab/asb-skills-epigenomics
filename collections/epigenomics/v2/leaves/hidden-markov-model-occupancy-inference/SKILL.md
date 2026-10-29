---
name: hidden-markov-model-occupancy-inference
description: Use when you have paired-end MNase-seq or ATAC-seq reads plus genome
  sequence for S. cerevisiae and need probabilistic nucleosome/TF occupancy scores
  or dyad positions across the genome.
when_to_use_negative:
- You only need peak calling or footprint identification plus motif labeling — the
  article contrasts RoboCOP with a MACS2 peak/footprint identification followed by
  FIMO TF-labeling pipeline, which performs notably worse
- You need a method that jointly models many factors from multiple data types beyond
  MNase-seq/ATAC-seq accessibility — RoboCOP currently uses only MNase-seq or ATAC-seq
- You require versioned, quantified sequencing-depth guidance — the article does not
  state depth requirements beyond noting accuracy improves with deeper sequencing
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3176
- http://edamontology.org/topic_0749
tools:
- name: RoboCOP
  role: Multivariate HMM that integrates nucleotide sequence with MNase-seq or ATAC-seq
    accessibility data to compute the probabilistic nucleosome/TF occupancy landscape;
    compiled via bash gccCompile and run through the example_robocop.ipynb vignette
  repo: https://github.com/HarteminkLab/RoboCOP
- name: NucleoATAC
  role: Benchmark alternative for nucleosome positioning from ATAC-seq; RoboCOPATAC
    positions many more nucleosomes than NucleoATAC using the same data
- name: MACS2
  role: Alternative peak/footprint identification approach (followed by FIMO TF-labeling)
    used as a comparison baseline
- name: FIMO
  role: TF-labeling step in the alternative MACS2-based pipeline used for comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:86521d07d9f04bb8f5cf3f2ad6518b48bdbd71ac9ae6d863b2f86f7aa90b7827
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - FIMO
  - MACS2
  - NucleoATAC
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3176
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# hidden-markov-model-occupancy-inference

## Summary

Run RoboCOP, a multivariate HMM that integrates chromatin accessibility data (paired-end MNase-seq or ATAC-seq) with nucleotide sequence, to compute a genome-wide probabilistic occupancy landscape of nucleosomes and transcription factors at single-nucleotide resolution.

## When to use

Use when you have paired-end MNase-seq or ATAC-seq reads plus genome sequence for S. cerevisiae and need probabilistic nucleosome/TF occupancy scores or dyad positions across the genome.

## When NOT to use

- You only need peak calling or footprint identification plus motif labeling — the article contrasts RoboCOP with a MACS2 peak/footprint identification followed by FIMO TF-labeling pipeline, which performs notably worse
- You need a method that jointly models many factors from multiple data types beyond MNase-seq/ATAC-seq accessibility — RoboCOP currently uses only MNase-seq or ATAC-seq
- You require versioned, quantified sequencing-depth guidance — the article does not state depth requirements beyond noting accuracy improves with deeper sequencing

## Inputs

- Paired-end MNase-seq data for S. cerevisiae (e.g. GEO accession GSE153609)
- ATAC-seq data (e.g. GEO accession GSE66386) as an alternative accessibility input
- Nucleotide (reference genome) sequence
- RoboCOP configuration file (config.ini) with cshared path to librobocop.so

## Outputs

- Genome-wide probabilistic occupancy scores of nucleosomes and up to 150 TFs at single-nucleotide resolution
- Called nucleosome dyad positions extracted from the inferred occupancy profile
- Precision-recall evaluation of dyad predictions against chemical-cleavage dyad annotations (e.g. robocop_mnase_pr_curve.png)

## How to apply

Download the paired-end MNase-seq data (e.g. S. cerevisiae from GEO accession GSE153609), clone the RoboCOP repository, and compile the shared library with 'bash gccCompile' to produce librobocop.so, whose absolute path must be set as cshared in config.ini (following config_example.ini). A conda environment with the required Python and R modules can be created from robocop-spec-file.txt via 'conda create --name robocop-env --file robocop-spec-file.txt'. Run RoboCOP in MNase-seq-only mode following the example_robocop.ipynb vignette so the multivariate HMM infers genome-wide probabilistic nucleosome occupancy, then extract called nucleosome dyad positions from the inferred occupancy profile. Validate positioning by scoring each called dyad as a true positive if it lies within 50 bp of a high-precision chemical-cleavage (Brogaard et al.) dyad annotation, and compute precision and recall over all called dyads; the dyad probability score should peak precisely at annotated dyads.

## Related tools

- **RoboCOP** (Multivariate HMM that integrates nucleotide sequence with MNase-seq or ATAC-seq accessibility data to compute the probabilistic nucleosome/TF occupancy landscape; compiled via bash gccCompile and run through the example_robocop.ipynb vignette) — https://github.com/HarteminkLab/RoboCOP
- **NucleoATAC** (Benchmark alternative for nucleosome positioning from ATAC-seq; RoboCOPATAC positions many more nucleosomes than NucleoATAC using the same data)
- **MACS2** (Alternative peak/footprint identification approach (followed by FIMO TF-labeling) used as a comparison baseline)
- **FIMO** (TF-labeling step in the alternative MACS2-based pipeline used for comparison)

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git; cd RoboCOP/robocop; bash gccCompile; conda create --name robocop-env --file robocop-spec-file.txt
```

## Evaluation signals

- Called dyads are scored as true positives when within 50 bp of a Brogaard et al. chemical-cleavage dyad annotation; precision and recall computed over all called dyads and plotted as a PR curve (Figure 4C, Supplementary Figure S12A)
- RoboCOP's dyad probability score peaks precisely at annotated dyads from reference (24), confirming alignment with ground-truth nucleosome positions
- librobocop.so exists after running 'bash gccCompile' and its absolute path is correctly set as cshared in config.ini
- MNase-seq-based runs should show markedly better performance than ATAC-seq-based runs, consistent with the article's observation
- RoboCOP with MNase- or ATAC-seq data should outperform the MACS2 + FIMO alternative baseline

## Limitations

- No version numbers are stated for any tool (RoboCOP, COMPETE, NucleoATAC, MACS2, FIMO)
- Sequencing depth requirements are not quantified; the article notes accuracy and comprehensiveness of chromatin occupancy profiles improve with deeper sequencing
- Prior information about DBF transition probabilities is not incorporated (deferred to future work)
- RoboCOP currently uses only MNase-seq or ATAC-seq accessibility data

## Evidence
