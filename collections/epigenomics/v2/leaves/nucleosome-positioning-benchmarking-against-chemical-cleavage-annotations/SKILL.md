---
name: nucleosome-positioning-benchmarking-against-chemical-cleavage-annotations
description: Use when a genome-wide nucleosome positioning prediction (dyad calls
  or occupancy-derived dyads) must be validated quantitatively against gold-standard
  chemical-cleavage dyad annotations, e.g. reproducing Figure 4C.
when_to_use_negative:
- Validating transcription factor (DBF) binding site predictions, which are benchmarked
  against ChIP-chip, ChIP-exo, and ORGANIC annotations rather than chemical-cleavage
  dyads
- The input is raw MNase-seq reads with no dyad predictions extracted yet — first
  run RoboCOP occupancy inference before benchmarking dyads
- Benchmarking ATAC-seq-based predictions against MNase-specific dyad score expectations,
  since the article reports markedly better performance with MNase-seq than ATAC-seq
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3176
- http://edamontology.org/topic_0749
tools:
- name: RoboCOP
  role: Multivariate HMM that infers genome-wide probabilistic nucleosome occupancy
    from MNase-seq and nucleotide sequence; its dyad calls are the predictions being
    benchmarked
  repo: https://github.com/HarteminkLab/RoboCOP
- name: gccCompile
  role: Shell script compiled via 'bash gccCompile' to build the librobocop.so shared
    library required by RoboCOP before running the occupancy model
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:953cfb5c01b3b97d503e899e2b067f2e86fad2ba5a0c6c45edf4e5d2b2caf53c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - RoboCOP
  - gccCompile
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3176
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# nucleosome-positioning-benchmarking-against-chemical-cleavage-annotations

## Summary

Benchmarks computationally predicted nucleosome dyad positions (e.g., from RoboCOP_MNase) against high-precision chemical-cleavage dyad annotations (Brogaard et al., ref 24) using precision-recall analysis over a 50 bp match tolerance.

## When to use

Use when a genome-wide nucleosome positioning prediction (dyad calls or occupancy-derived dyads) must be validated quantitatively against gold-standard chemical-cleavage dyad annotations, e.g. reproducing Figure 4C.

## When NOT to use

- Validating transcription factor (DBF) binding site predictions, which are benchmarked against ChIP-chip, ChIP-exo, and ORGANIC annotations rather than chemical-cleavage dyads
- The input is raw MNase-seq reads with no dyad predictions extracted yet — first run RoboCOP occupancy inference before benchmarking dyads
- Benchmarking ATAC-seq-based predictions against MNase-specific dyad score expectations, since the article reports markedly better performance with MNase-seq than ATAC-seq

## Inputs

- Paired-end MNase-seq data for S. cerevisiae (GEO accession GSE153609)
- High-precision chemical-cleavage nucleosome dyad annotations (Brogaard et al., reference 24)
- S. cerevisiae genome sequence
- RoboCOP configuration file (config.ini) with cshared path to librobocop.so

## Outputs

- Genome-wide probabilistic nucleosome occupancy profile from the RoboCOP multivariate HMM
- Called nucleosome dyad positions
- Precision-recall curve for RoboCOP_MNase dyad predictions (reproducing Figure 4C and Supplementary Figure S12A), e.g. saved as robocop_mnase_pr_curve.png
- Precision and recall values at a 50 bp dyad-matching tolerance

## How to apply

Download paired-end MNase-seq data for S. cerevisiae from GEO accession GSE153609, clone github.com/HarteminkLab/RoboCOP, compile the shared library with 'bash gccCompile' to produce librobocop.so, and set its absolute path as 'cshared' in config.ini (per config_example.ini). Run RoboCOP in MNase-seq-only mode following the example_robocop.ipynb vignette so the multivariate HMM infers genome-wide probabilistic nucleosome occupancy. Extract called dyad positions from the occupancy profile, score each called dyad a true positive if it falls within 50 bp of a chemical-cleavage (Brogaard et al., 24) annotated dyad, and compute precision and recall over all called dyads while varying the calling threshold. Plot the precision-recall curve; success is a dyad probability score that peaks precisely at the annotated dyads, as in Figure 4C and Supplementary Figure S12A.

## Related tools

- **RoboCOP** (Multivariate HMM that infers genome-wide probabilistic nucleosome occupancy from MNase-seq and nucleotide sequence; its dyad calls are the predictions being benchmarked) — https://github.com/HarteminkLab/RoboCOP
- **gccCompile** (Shell script compiled via 'bash gccCompile' to build the librobocop.so shared library required by RoboCOP before running the occupancy model) — https://github.com/HarteminkLab/RoboCOP

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git; cd RoboCOP/robocop; bash gccCompile; jupyter notebook ../analysis/example_robocop.ipynb
```

## Evaluation signals

- RoboCOP_MNase dyad probability score peaks precisely at the annotated chemical-cleavage dyads from reference (24)
- Precision-recall curve is generated for RoboCOP_MNase matching Figure 4C and Supplementary Figure S12A
- librobocop.so exists at the path set in config.ini ('ls "`pwd`/librobocop.so"' returns the file)
- Each called dyad is scored a true positive only if within 50 bp of an annotated dyad; precision and recall computed over all called dyads are finite and threshold-dependent
- Predicted dyads derive from the MNase-seq-only mode run following example_robocop.ipynb, not from a mixed MNase/ATAC-seq run

## Limitations

- No version numbers are stated for RoboCOP or any comparison tool, so exact reproducibility may vary with repository state
- Performance is data-type dependent: the article observes markedly better performance with MNase-seq than ATAC-seq, so benchmark results do not transfer directly across assays
- Accuracy of the occupancy profile improves with deeper sequencing; sequencing depth requirements are not quantified

## Evidence

- [methods] Score each called dyad as a true positive if it lies within 50 bp of a dyad from the high-precision chemical cleavage (24) annotations, and compute precision and recall over all called dyads.: [evidence span withheld — anchor status: not_found]
