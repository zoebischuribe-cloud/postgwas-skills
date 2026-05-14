# Colocalization Hypotheses Reference

## The Five Hypotheses

coloc tests five mutually exclusive hypotheses about the relationship between two traits at a locus:

### H0: No Association
- Neither trait is associated with the locus
- PP0: Posterior probability of H0
- Interpretation: The locus is not associated with either trait

### H1: Association with Trait 1 Only
- Only trait 1 (e.g., GWAS) is associated
- PP1: Posterior probability of H1
- Interpretation: GWAS signal exists, but no QTL signal

### H2: Association with Trait 2 Only
- Only trait 2 (e.g., QTL) is associated
- PP2: Posterior probability of H2
- Interpretation: QTL signal exists, but no GWAS signal

### H3: Both Associated, Different Causal Variants
- Both traits are associated, but with different causal variants
- PP3: Posterior probability of H3
- Interpretation: GWAS and QTL signals are distinct (possibly due to LD)

### H4: Both Associated, Same Causal Variant
- Both traits are associated with the **same** causal variant
- PP4: Posterior probability of H4
- Interpretation: **Colocalization** - the GWAS signal is likely mediated through the QTL

---

## Decision Framework

```
PP4 ≥ 0.8?
├── Yes → Strong evidence of colocalization
│          → Prioritize gene for follow-up
│
└── No → PP3 > PP4?
         ├── Yes → Different causal variants
         │          → Not colocalized
         │
         └── No → PP1 or PP2 high?
                  ├── PP1 high → GWAS only
                  └── PP2 high → QTL only
```

---

## Prior Settings

### Default Priors
- p1 = 1e-4 (prior probability a SNP is associated with trait 1)
- p2 = 1e-4 (prior probability a SNP is associated with trait 2)
- p12 = 1e-5 (prior probability a SNP is associated with both traits)

### Adjusting Priors

For rare diseases:
```r
result <- coloc.abf(
  dataset1, dataset2,
  p1 = 1e-5,  # Lower prior for rare disease
  p2 = 1e-4,
  p12 = 1e-6
)
```

For well-powered studies:
```r
result <- coloc.abf(
  dataset1, dataset2,
  p1 = 1e-3,  # Higher prior for well-powered
  p2 = 1e-3,
  p12 = 1e-4
)
```

---

## Sensitivity Analysis

### Why Run Sensitivity?
- Test robustness to prior assumptions
- Identify borderline cases
- Understand impact of sample size

### Approach

```r
# Test range of priors
p12_values <- c(1e-6, 1e-5, 1e-4)

for (p12 in p12_values) {
  result <- coloc.abf(trait1, trait2, p12 = p12)
  print(paste("p12 =", p12, "PP4 =", result$summary[5]))
}
```

---

## Multiple Causal Variants

When there may be multiple causal variants at a locus, use SuSiE-based coloc:

```r
library(susieR)

# Run SuSiE coloc
result <- coloc.susie(trait1, trait2)

# Results include multiple signals
# Each signal has its own PP4
```

### Interpretation
- Multiple PP4 values → multiple colocalization events
- Sum of PP4s → overall colocalization probability
