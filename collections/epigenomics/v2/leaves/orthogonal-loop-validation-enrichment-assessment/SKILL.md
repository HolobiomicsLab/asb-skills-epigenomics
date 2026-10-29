---
name: orthogonal-loop-validation-enrichment-assessment
description: Use when you have a 10 kb-binned .cool/.hic Hi-C matrix (e.g., GM12878)
  and need to benchmark predicted loops against enrichment-based callers or validate
  loop sets against orthogonal data such as H3K27ac HiChIP or ENCODE ChIP-Seq peaks.
when_to_use_negative:
- 'Very low sequencing depth (e.g., 1.5% down-sample rate): Fit-Hi-C yields no interactions
  at q < 1e-5 and the cutoff must be relaxed to 0.1, so the standard comparison is
  not reproducible'
- Enrichment-only questions where supervised benchmarking is unnecessary (e.g., you
  only need one caller's loops without overlap/validation assessment)
- Platforms lacking dedicated loop-detection algorithms or compatible formats, such
  as DNA SPRITE matrices
tools:
- name: Peakachu
  role: Supervised Random Forest loop caller generating the genome-wide predicted
    loop set via score_genome + pool at probability 0.9-0.95
  repo: https://github.com/tariks/peakachu
- name: HiCCUPS
  role: Juicer enrichment-based loop caller using local background and a Poisson test,
    providing the comparison loop set
  repo: https://github.com/aidenlab/juicer
- name: Fit-Hi-C
  role: Distance-dependent two-pass spline caller; second-pass output filtered at
    q-value < 1e-5 and top 140,000 interactions pooled
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:2698dd315766a52f29110764fa66eaf9eec05f000eea7bbd1ab53d9c635f96fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Fit-Hi-C
  - HiCCUPS
  - Peakachu
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# orthogonal-loop-validation-enrichment-assessment

## Summary

Compares supervised (Peakachu) chromatin loop calls against enrichment-based methods (HiCCUPS, Fit-Hi-C) on the same 10 kb Hi-C matrix, building a three-way Venn overlap and computing method-specific validation ratios against orthogonal ChIA-PET/HiChIP/ChIP-Seq evidence. It quantifies that Peakachu recovers a unique, well-validated loop set (72% overlap, 58% vs 48%/40% validation).

## When to use

Use when you have a 10 kb-binned .cool/.hic Hi-C matrix (e.g., GM12878) and need to benchmark predicted loops against enrichment-based callers or validate loop sets against orthogonal data such as H3K27ac HiChIP or ENCODE ChIP-Seq peaks.

## When NOT to use

- Very low sequencing depth (e.g., 1.5% down-sample rate): Fit-Hi-C yields no interactions at q < 1e-5 and the cutoff must be relaxed to 0.1, so the standard comparison is not reproducible
- Enrichment-only questions where supervised benchmarking is unnecessary (e.g., you only need one caller's loops without overlap/validation assessment)
- Platforms lacking dedicated loop-detection algorithms or compatible formats, such as DNA SPRITE matrices

## Inputs

- 10 kb-resolution .cool file (e.g., Rao2014-GM12878-MboI-allreps-filtered.10kb.cool)
- Peakachu-trained random forest models per chromosome (from 'peakachu train')
- Hi-C .hic or .cool matrix for Juicer/HiCCUPS
- Fit-Hi-C input interaction list at 10 kb binning
- Orthogonal validation sets (e.g., gm12878.mumbach.h3k27ac-hichip.hg19.bedpe, ENCODE ChIP-Seq peak files for 133 TFs and 10 histone modifications in GM12878)

## Outputs

- Three-way Venn overlap summary of Peakachu/HiCCUPS/Fit-Hi-C loop sets
- Method-specific loop lists (Peakachu pool output, HiCCUPS loop list, Fit-Hi-C top-140,000 interactions)
- method_comparison_summary.tsv with overlap percentage (72%) and validation ratios (58%/48%/40%)
- Fold-enrichment validation statistics at loop anchors with 50 shuffled controls

## How to apply

Load the Rao2014-GM12878 10 kb cooler (Rao2014-GM12878-MboI-allreps-filtered.10kb.cool) and generate the Peakachu genome-wide loop set with 'peakachu score_genome' followed by 'peakachu pool' at a probability threshold of 0.9-0.95, which selects the most significant non-redundant results from per-pixel probabilities. Run HiCCUPS (Juicer) on the same matrix; note that HiCCUPS incorporates local background via a Poisson test. Run Fit-Hi-C's distance-dependent two-pass spline, filter the second pass at q-value < 1e-5, sort surviving interactions by p-value, and pool the top 140,000 interactions. Intersect the three loop sets to build the Venn overlap (reporting Peakachu's 72% overlap with either enrichment-based method), then compute fold-enrichment validation of method-specific loops against orthogonal sets, using 50 shuffled loop controls for the validation procedure. Accept the result if the 72% overlap and the 58%/48%/40% validation ratios for Peakachu-/HiCCUPS-/Fit-Hi-C-specific loops are reproduced.

## Related tools

- **Peakachu** (Supervised Random Forest loop caller generating the genome-wide predicted loop set via score_genome + pool at probability 0.9-0.95) — https://github.com/tariks/peakachu
- **HiCCUPS** (Juicer enrichment-based loop caller using local background and a Poisson test, providing the comparison loop set) — https://github.com/aidenlab/juicer
- **Fit-Hi-C** (Distance-dependent two-pass spline caller; second-pass output filtered at q-value < 1e-5 and top 140,000 interactions pooled)

## Examples

```
peakachu score_genome -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool --clr-weight-name weight -O scores.txt; peakachu pool -O peakachu_loops.txt --minimum-prob 0.95
```

## Evaluation signals

- The three-way Venn overlap reproduces 72% (12,398/17,171) of Peakachu loops overlapping HiCCUPS or Fit-Hi-C predictions
- Method-specific validation ratios match 58% (Peakachu-specific), 48% (HiCCUPS-specific), and 40% (Fit-Hi-C-specific)
- Fit-Hi-C second-pass output is filtered at q-value < 1e-5 and the top 140,000 interactions by p-value are pooled
- Peakachu loop set comes from score_genome + pool at probability threshold 0.9-0.95 with non-redundant centroid loci
- Validation fold-enrichments are accompanied by 50 shuffled-loop controls per comparison

## Limitations

- Each method (Peakachu, HiCCUPS, Fit-Hi-C) detects a unique set of chromatin loops with different distance distributions, CTCF binding, and validation ratios, so no single method captures the complete loop set
- HiCCUPS with default parameters outputted less than 1/3 of the Peakachu loops at each down-sample rate; at 1.5% depth parameters had to be tuned to -f 0.9 -t 3.6,1,1,1
- At the 1.5% down-sample rate, no interactions remained for Fit-Hi-C at q-value < 1e-5, requiring a relaxed cutoff of 0.1

## Evidence

- [discussion] each method detects a unique set of chromatin loops with various distance distributions, CTCF binding, and validation ratios: "each method detects a unique set of chromatin loops with various distance distributions, CTCF binding, and validation ratios"
- [discussion] Fit-Hi-C q-value filter: "The results of the 2nd spline pass were then filtered with the q-value cutoff < 1e-5"
- [intro] HiCCUPS model description: "HiCCUPS8,23 incorporates local background into its model and utilizes the Poisson test"
- [methods] pool function description: "The pool function serves to select the most significant non-redundant results from per-pixel probabilities calculated by the score functions"
- [discussion] randomly shuffled loop controls: "we randomly shuffled the loops to generate 50 controls and repeated the same procedure for each control."
