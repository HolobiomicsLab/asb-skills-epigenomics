---
name: pretrained-random-forest-loop-scoring
description: Use when a .cool/.hic Hi-C or HiChIP contact map needs loop calling with
  a pre-trained Peakachu model, or when an FDR estimate for loop predictions is required
  via a perturbation expected to abolish loops.
when_to_use_negative:
- You lack a perturbation condition in which loops are biologically expected to disappear,
  so no FDR estimate can be grounded
- The contact map is not binned at ~10 kb or lacks the resolution needed by the 10
  kb pre-trained model
- You need de novo training rather than scoring — use peakachu train with a bedpe
  positive set instead
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: Peakachu
  role: Random forest framework whose score_genome and pool subcommands score per-pixel
    loop probabilities with a pre-trained CTCF ChIA-PET model and select significant
    non-redundant loops
  repo: https://github.com/tariks/peakachu
- name: cooler
  role: Loads the .mcool/.cool RAD21-AID Hi-C contact maps at 10 kb resolution for
    scoring
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:afd167065b4de0a6200c9bd5441735f96a7c4a830fd1a23a473d65dce0a83ae1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Peakachu
  - cooler
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# pretrained-random-forest-loop-scoring

## Summary

Apply Peakachu's pre-trained CTCF ChIA-PET random forest models to score per-pixel loop probabilities in a 10 kb-resolution Hi-C contact map, pool significant non-redundant loops, and estimate model false discovery rate using a cohesin-depletion perturbation (RAD21-AID auxin treatment) where loops are expected to disappear.

## When to use

Use when a .cool/.hic Hi-C or HiChIP contact map needs loop calling with a pre-trained Peakachu model, or when an FDR estimate for loop predictions is required via a perturbation expected to abolish loops.

## When NOT to use

- You lack a perturbation condition in which loops are biologically expected to disappear, so no FDR estimate can be grounded
- The contact map is not binned at ~10 kb or lacks the resolution needed by the 10 kb pre-trained model
- You need de novo training rather than scoring — use peakachu train with a bedpe positive set instead

## Inputs

- .mcool or .cool Hi-C contact map at 10 kb resolution (e.g. auxin-treated and untreated HCT-116 RAD21-AID Hi-C)
- Pre-trained Peakachu random forest model pickle (e.g. CTCCF-ChIAPET-peakachu-pretrained.*.10kb.pkl)
- Probability threshold for pooling (e.g. -t 0.95)

## Outputs

- Per-pixel loop probability bedpe files from score_genome
- Pooled, non-redundant loop lists (centroid loci) per condition
- Loop counts per condition and an estimated FDR (~0.2% = 19/11,814)

## How to apply

Load the auxin-treated and untreated HCT-116 RAD21-AID Hi-C contact maps as .mcool/.cool files at 10 kb resolution using cooler. Run 'peakachu score_genome -r 10000' on each map with the CTCF ChIA-PET pre-trained random forest model (CTCF-ChIAPET-peakachu-pretrained.*.10kb.pkl), setting --clr-weight-name to raw because CTCF ChIA-PET models were trained on raw contact values. Then run 'peakachu pool' on each per-pixel score bedpe with a probability threshold (e.g. -t 0.95) to select the most significant non-redundant loops. Count pooled loops in both conditions: the auxin-treated map should yield ~19 loops versus ~11,814 in untreated at similar sequencing depth, giving an estimated FDR of ~0.2% (19/11,814) since loops are expected to disappear upon cohesin depletion.

## Related tools

- **Peakachu** (Random forest framework whose score_genome and pool subcommands score per-pixel loop probabilities with a pre-trained CTCF ChIA-PET model and select significant non-redundant loops) — https://github.com/tariks/peakachu
- **cooler** (Loads the .mcool/.cool RAD21-AID Hi-C contact maps at 10 kb resolution for scoring)

## Examples

```
peakachu score_genome -r 10000 -p auxin-treated.RAD21-AID.mcool::/10 --clr-weight-name raw -m CTCF-ChIAPET-peakachu-pretrained.ctcf.10kb.pkl -O auxin.scores.bedpe; peakachu pool -t 0.95 -i auxin.scores.bedpe
```

## Evaluation signals

- Auxin-treated (cohesin-depleted) map yields ~19 pooled loops while untreated map of similar sequencing depth yields ~11,814 loops
- Estimated FDR of ~0.2% (19/11,814) is reproduced from the loop counts
- score_genome is run with --clr-weight-name raw, matching the raw-contact training scheme of CTCF ChIA-PET models
- pool is applied with a probability threshold (e.g. -t 0.95) and outputs non-redundant centroid loci rather than raw per-pixel scores

## Limitations

- Peakachu models were trained on raw contact values for CTCF ChIA-PET, so normalized weight columns must not be substituted without justification
- Probability cutoffs are model- and cell-line-specific (e.g. 0.97 for CTCF, 0.92 for H3K27ac in GM12878); other cutoffs are relegated to Supplementary Data 2
- Peakachu detects a unique set of loops distinct from HiCCUPS and Fit-Hi-C, so no single method captures the complete loop set

## Evidence

- [other] CTCF ChIA-PET models were trained on raw contact values: [evidence span withheld — anchor status: not_found]
- [other] peakachu identified 19 loops in auxin-treated versus 11,814 in untreated: [evidence span withheld — anchor status: not_found]
- [methods] pool selects most significant non-redundant results: "The pool function serves to select the most significant non-redundant results from per-pixel probabilities calculated by the score functions"
- [readme] Peakachu supports .hic and .cool formats: "Peakachu (version>=1.1.2) now supports both [.hic](https://github.com/aidenlab/juicer/wiki/Data) and [.cool](https://cooler.readthedocs.io/en/latest/datamodel.html) formats."
