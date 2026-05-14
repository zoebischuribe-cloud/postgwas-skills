# pgwas-mr: Mendelian Randomization Skill

**Status**: Beta  
**Purpose**: Two-sample Mendelian Randomization workflow for causal inference  
**Trigger keywords**: "MR", "Mendelian randomization", "causal inference", "two-sample MR"

---

## What it does

Performs complete two-sample Mendelian Randomization analysis from GWAS summary statistics, including:

1. **Instrument selection** - LD clumping, p-value thresholding
2. **Data harmonization** - Allele alignment, remove incompatible SNPs
3. **MR analysis** - IVW, MR-Egger, weighted median, weighted mode
4. **Sensitivity analysis** - Heterogeneity, pleiotropy, leave-one-out
5. **Visualization** - Scatter, forest, funnel plots
6. **Reporting** - Formatted results summary

## Quick Start

```r
library(TwoSampleMR)

# 1. Load exposure data
exposure <- read_exposure_data("exposure_gwas.txt", ...)

# 2. Clump SNPs
exposure_clumped <- clump_data(exposure)

# 3. Extract outcome data
outcome <- extract_outcome_data(exposure_clumped$SNP, "ieu-a-2")

# 4. Harmonize
harmonised <- harmonise_data(exposure_clumped, outcome)

# 5. Run MR
results <- mr(harmonised)

# 6. Sensitivity
mr_pleiotropy_test(harmonised)
mr_heterogeneity(harmonised)
```

## Methods Implemented

| Method | Assumption | Robustness |
|--------|------------|------------|
| IVW | No pleiotropy | Standard |
| MR-Egger | InSIDE | Detects pleiotropy |
| Weighted Median | <50% invalid | Robust |
| Weighted Mode | Modal valid | Very robust |

## Output Files

- `mr_results.csv` - Main MR estimates
- `mr_scatter.pdf` - Scatter plot with regression lines
- `mr_forest.pdf` - Forest plot of single-SNP estimates
- `mr_funnel.pdf` - Funnel plot for asymmetry detection
- `mr_report.md` - Formatted results summary

## Dependencies

- R >= 4.0
- TwoSampleMR package
- MRPRESSO package (optional)
- ggplot2

## References

- Hemani G, et al. (2018) The MR-Base platform supports systematic causal inference. *Nature Genetics*
- Bowden J, et al. (2015) Mendelian randomization with invalid instruments. *Int J Epidemiol*
