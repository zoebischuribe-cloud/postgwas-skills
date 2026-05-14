# pgwas-coloc: Colocalization Analysis Skill

**Status**: Beta  
**Purpose**: Assess whether two traits share the same causal variant  
**Trigger keywords**: "colocalization", "coloc", "shared causal variant", "co-localization"

---

## What it does

Performs colocalization analysis to determine if GWAS and QTL signals share the same causal variant, including:

1. **Regional data extraction** - Define and extract summary statistics
2. **Colocalization test** - Bayesian analysis with coloc package
3. **Result interpretation** - PP4 posterior probability assessment
4. **Visualization** - Locus comparison plots
5. **Reporting** - Formatted results summary

## Quick Start

```r
library(coloc)

# Prepare data
trait1 <- list(
  pvalues = gwas$pval,
  N = gwas_n,
  MAF = gwas$eaf,
  type = "quant"
)

trait2 <- list(
  pvalues = qtl$pval,
  N = qtl_n,
  MAF = qtl$eaf,
  type = "quant"
)

# Run coloc
result <- coloc.abf(trait1, trait2)

# Check PP4
result$summary[5]  # PP4 (shared causal variant)
```

## Colocalization Hypotheses

| Hypothesis | Meaning | PP |
|------------|---------|-----|
| H0 | No association | PP0 |
| H1 | Trait 1 only | PP1 |
| H2 | Trait 2 only | PP2 |
| H3 | Different variants | PP3 |
| H4 | **Same variant** | PP4 |

**Decision**: PP4 ≥ 0.8 → Strong evidence of colocalization

## Output Files

- `coloc_results.csv` - Posterior probabilities
- `locus_compare.pdf` - Locus comparison plot
- `coloc_report.md` - Formatted results summary

## Dependencies

- R >= 4.0
- coloc package
- susieR package (optional)
- ggplot2

## References

- Giambartolomei C, et al. (2014) Bayesian colocalization. *PLoS Genet*
- Wallace C (2021) A more accurate method for colocalisation. *Int J Epidemiol*
