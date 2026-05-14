---
name: pgwas-mr
description: Mendelian Randomization analysis workflow for causal inference from GWAS summary statistics. Use when user mentions MR, Mendelian randomization, causal inference, two-sample MR, IVW, MR-Egger, weighted median, or wants to estimate causal effect of exposure on outcome.
---

# pgwas-mr: Mendelian Randomization Workflow

A comprehensive workflow for performing two-sample Mendelian Randomization analysis to estimate causal effects from GWAS summary statistics.

## When to Use This Skill

Trigger this skill when the user:
- Mentions "MR", "Mendelian randomization", "causal inference"
- Wants to estimate causal effect of an exposure on an outcome
- Has GWAS summary statistics and wants to do causal analysis
- Mentions specific MR methods (IVW, MR-Egger, weighted median, MR-PRESSO)
- Asks about instrument selection, harmonization, or sensitivity analysis

## Workflow Overview

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  Instrument     │───▶│  Data            │───▶│  MR Analysis    │
│  Selection      │    │  Harmonization   │    │  & Sensitivity  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
        │                      │                       │
        ▼                      ▼                       ▼
   LD clumping          Align alleles            IVW, Egger
   P-value threshold    Remove palindromic       Weighted median
   MAF filter           SNPs                     MR-PRESSO
```

## Step 1: Instrument Selection

### Input Requirements
- Exposure GWAS summary statistics (CSV/TSV)
- Required columns: `SNP`, `beta`, `se`, `pval`, `eaf` (effect allele frequency), `effect_allele`, `other_allele`

### Selection Criteria
1. **P-value threshold**: Default `p < 5e-8` for genome-wide significant SNPs
2. **LD clumping**: Remove correlated SNPs (r² < 0.01 within 1 Mb window)
3. **MAF filter**: Exclude SNPs with MAF < 0.01 (optional)
4. **Remove HLA region**: Chr6:25-34 Mb (optional, recommended)

### Code Template (R)

```r
library(TwoSampleMR)

# Format exposure data
exposure_dat <- read_exposure_data(
  filename = "exposure_gwas.txt",
  sep = "\t",
  snp_col = "SNP",
  beta_col = "beta",
  se_col = "se",
  effect_allele_col = "effect_allele",
  other_allele_col = "other_allele",
  eaf_col = "eaf",
  pval_col = "pval"
)

# LD clumping
exposure_clumped <- clump_data(
  exposure_dat,
  clump_kb = 1000,
  clump_r2 = 0.01,
  clump_p1 = 1,
  clump_p2 = 1
)
```

## Step 2: Outcome Data Extraction

### Options for Outcome Data

1. **IEU GWAS database** (pre-formatted):
```r
outcome_dat <- extract_outcome_data(
  snps = exposure_clumped$SNP,
  outcomes = "ieu-a-2",  # e.g., BMI GWAS
  proxies = TRUE
)
```

2. **Local GWAS file**:
```r
outcome_dat <- read_outcome_data(
  filename = "outcome_gwas.txt",
  sep = "\t",
  snp_col = "SNP",
  beta_col = "beta",
  se_col = "se",
  effect_allele_col = "effect_allele",
  other_allele_col = "other_allele",
  eaf_col = "eaf",
  pval_col = "pval"
)
```

### Common Outcome GWAS IDs (IEU Database)

| Trait | ID | Sample Size |
|-------|-----|-------------|
| BMI | ieu-a-2 | 322,154 |
| Height | ieu-a-812 | 252,901 |
| Type 2 Diabetes | ebi-a-GCST006867 | 74,124 |
| Coronary Heart Disease | ieu-a-7 | 184,305 |
| Alzheimer's | ieu-a-237 | 63,826 |

## Step 3: Harmonization

Harmonize exposure and outcome data to ensure consistent effect alleles.

```r
# Harmonize data
harmonised_dat <- harmonise_data(
  exposure_dat = exposure_clumped,
  outcome_dat = outcome_dat,
  action = 2  # Remove incompatible SNPs
)

# Check harmonization summary
print(harmonised_dat)
```

### Harmonization Actions
- `action = 1`: Assume all alleles are correct (not recommended)
- `action = 2`: Remove incompatible SNPs (default)
- `action = 3`: Try to infer strand orientation

## Step 4: MR Analysis

### Main MR Methods

```r
# Perform MR
mr_results <- mr(harmonised_dat)

# Default methods:
# - IVW (Inverse Variance Weighted)
# - MR-Egger
# - Weighted Median
# - Weighted Mode
```

### Method Selection Guide

| Method | Assumption | When to Use |
|--------|------------|-------------|
| IVW | No horizontal pleiotropy | Primary method, most power |
| MR-Egger | InSIDE assumption | Detect directional pleiotropy |
| Weighted Median | <50% invalid instruments | Robust to outliers |
| Weighted Mode | Largest valid subset | Robust to heterogeneous effects |

### Single SNP Analysis

```r
# Single SNP MR
mr_singlesnp <- mr_singlesnp(harmonised_dat)

# Forest plot
p1 <- mr_forest_plot(mr_singlesnp)
ggsave("mr_forest.pdf", p1)
```

## Step 5: Sensitivity Analysis

### Pleiotropy Test

```r
# MR-Egger intercept test
egger_intercept <- mr_pleiotropy_test(harmonised_dat)
```

- Significant intercept → directional pleiotropy present
- Report intercept p-value and interpretation

### Heterogeneity Test

```r
# Heterogeneity statistics
heterogeneity <- mr_heterogeneity(harmonised_dat)
```

- High heterogeneity (Q p-value < 0.05) → potential invalid instruments
- Consider removing outlier SNPs

### Leave-One-Out Analysis

```r
# Leave-one-out analysis
loo <- mr_leaveoneout(harmonised_dat)

# Plot
p2 <- mr_leaveoneout_plot(loo)
ggsave("mr_loo.pdf", p2)
```

### MR-PRESSO (Optional)

```r
library(MRPRESSO)

# Run MR-PRESSO
presso <- mr_presso(
  BetaOutcome = "outcome_beta",
  BetaExposure = "exposure_beta",
  SdOutcome = "outcome_se",
  SdExposure = "exposure_se",
  OUTLIERtest = TRUE,
  DISTORTIONtest = TRUE,
  data = harmonised_dat
)
```

## Step 6: Visualization

### Scatter Plot

```r
p3 <- mr_scatter_plot(mr_results, harmonised_dat)
ggsave("mr_scatter.pdf", p3[[1]])
```

### Funnel Plot

```r
p4 <- mr_funnel_plot(mr_singlesnp)
ggsave("mr_funnel.pdf", p4[[1]])
```

## Reporting Template

```markdown
## MR Analysis Results

### Instrument Selection
- Number of SNPs selected: X
- P-value threshold: 5e-8
- LD clumping: r² < 0.01, 1 Mb window

### Main Results
| Method | Beta | SE | P-value |
|--------|------|-----|---------|
| IVW | X.XX | X.XX | X.Xe-X |
| MR-Egger | X.XX | X.XX | X.Xe-X |
| Weighted Median | X.XX | X.XX | X.Xe-X |

### Sensitivity Analysis
- MR-Egger intercept: β = X.XX, p = X.XX
- Heterogeneity (IVW): Q = X.XX, p = X.XX
- Leave-one-out: No influential SNPs detected

### Conclusion
[Interpret results considering all sensitivity analyses]
```

## Common Issues & Solutions

### Issue 1: Too few instruments
- **Solution**: Relax p-value threshold (e.g., 1e-5) or use genome-wide data with weak instrument robust methods

### Issue 2: High heterogeneity
- **Solution**: Check for outlier SNPs, consider MR-PRESSO, or use weighted median/mode

### Issue 3: Palindromic SNPs
- **Solution**: Remove palindromic SNPs with MAF 0.3-0.7, or use allele frequency to infer strand

### Issue 4: Sample overlap
- **Solution**: Check for sample overlap between exposure and outcome; use simulation extrapolation if overlap exists

## Reference Files

For detailed information, see:
- `references/mr-methods.md` - Detailed method descriptions
- `references/gwas-databases.md` - Available GWAS databases

## Dependencies

```r
# Required R packages
install.packages("TwoSampleMR")
install.packages("MRPRESSO")
install.packages("ggplot2")
```

## Example Usage

**User**: "I want to do MR analysis to test if BMI causally affects type 2 diabetes"

**Response**: 
1. Load BMI GWAS as exposure (ieu-a-2)
2. Extract T2D GWAS as outcome (ebi-a-GCST006867)
3. Perform LD clumping on exposure SNPs
4. Harmonize exposure and outcome
5. Run IVW, MR-Egger, weighted median
6. Perform sensitivity analyses
7. Generate report and plots
