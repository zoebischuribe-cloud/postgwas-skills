---
name: pgwas-coloc
description: Colocalization analysis workflow to assess whether two traits share the same causal variant. Use when user mentions colocalization, coloc, shared causal variant, co-localization, or wants to test if GWAS and QTL signals overlap.
---

# pgwas-coloc: Colocalization Analysis Workflow

A comprehensive workflow for performing colocalization analysis to determine whether two traits (e.g., GWAS and eQTL) share the same causal variant at a locus.

## When to Use This Skill

Trigger this skill when the user:
- Mentions "colocalization", "coloc", "co-localization"
- Wants to test if GWAS and QTL signals share the same causal variant
- Asks about PP4, posterior probability, or coloc hypothesis
- Needs to prioritize genes based on eQTL colocalization
- Mentions "shared causal variant" or "same causal SNP"

## Workflow Overview

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  Regional       │───▶│  Colocalization  │───▶│  Result         │
│  Data Extract   │    │  Analysis        │    │  Interpretation │
└─────────────────┘    └──────────────────┘    └─────────────────┘
        │                      │                       │
        ▼                      ▼                       ▼
   Define region         coloc package          PP4 > 0.8?
   Extract summary       Bayesian test          Prioritize genes
   Align alleles         5 hypotheses          Locus report
```

## Colocalization Hypotheses

The coloc method tests 5 mutually exclusive hypotheses:

| Hypothesis | Description | Posterior Probability |
|------------|-------------|----------------------|
| H0 | No association with either trait | PP0 |
| H1 | Association with trait 1 only | PP1 |
| H2 | Association with trait 2 only | PP2 |
| H3 | Both traits associated, different causal variants | PP3 |
| H4 | Both traits associated, **same causal variant** | PP4 |

**Key metric**: PP4 (posterior probability of shared causal variant)

## Step 1: Define Genomic Region

### Input Requirements
- GWAS summary statistics for trait 1
- QTL summary statistics for trait 2 (eQTL, pQTL, etc.)
- Region to test (e.g., ±500 kb around lead SNP)

### Region Definition

```r
# Define region around lead SNP
lead_snp <- "rs12345"
chr <- 6
pos <- 30000000
region_size <- 500000  # ±500 kb

region <- c(
  chr = chr,
  start = pos - region_size,
  end = pos + region_size
)
```

### Gene-Centric Approach

For eQTL colocalization, test each gene's cis-window:

```r
# Gene information
gene <- "GENE_NAME"
gene_chr <- 6
gene_start <- 29500000
gene_end <- 30500000

# cis-window (typically ±1 Mb from gene boundary)
cis_window <- 1000000
region <- c(
  chr = gene_chr,
  start = gene_start - cis_window,
  end = gene_end + cis_window
)
```

## Step 2: Extract Regional Data

### Using coloc package

```r
library(coloc)

# Read GWAS data
gwas <- read.table("gwas_summary.txt", header = TRUE)
gwas_region <- subset(gwas, 
  CHR == region["chr"] & 
  POS >= region["start"] & 
  POS <= region["end"]
)

# Read QTL data
qtl <- read.table("eqtl_summary.txt", header = TRUE)
qtl_region <- subset(qtl,
  CHR == region["chr"] &
  POS >= region["start"] &
  POS <= region["end"]
)
```

### Using TwoSampleMR

```r
library(TwoSampleMR)

# Extract regional data from IEU database
gwas_region <- extract_outcome_data(
  snps = "chr6:29500000-30500000",
  outcomes = "ieu-a-2"
)
```

## Step 3: Prepare coloc Input

### Data Format

coloc requires:
- `pvalues` - P-values for each SNP
- `N` - Sample size
- `MAF` - Minor allele frequency
- `beta` (optional) - Effect sizes
- `varbeta` (optional) - Variance of effect sizes

### Code Template

```r
# Prepare trait 1 (GWAS)
trait1 <- list(
  pvalues = gwas_region$pval,
  N = gwas_n,  # GWAS sample size
  MAF = gwas_region$eaf,
  type = "quant",  # "quant" for quantitative, "cc" for case-control
  s = NULL  # proportion of cases (for case-control)
)

# Prepare trait 2 (QTL)
trait2 <- list(
  pvalues = qtl_region$pval,
  N = qtl_n,  # QTL sample size
  MAF = qtl_region$eaf,
  type = "quant"
)
```

### For Case-Control Traits

```r
# For binary trait (e.g., disease GWAS)
trait1 <- list(
  pvalues = gwas_region$pval,
  N = gwas_n,
  MAF = gwas_region$eaf,
  type = "cc",
  s = n_cases / gwas_n  # proportion of cases
)
```

## Step 4: Run Colocalization

### Basic coloc Analysis

```r
# Run coloc
result <- coloc.abf(
  dataset1 = trait1,
  dataset2 = trait2
)

# View results
print(result)
```

### Output Structure

```
Coloc analysis results:

PP.H0.abf  PP.H1.abf  PP.H2.abf  PP.H3.abf  PP.H4.abf
0.001      0.050      0.100      0.049      0.800

Best SNP by PP4:
rs12345 (PP4 = 0.85)
```

### Using coloc.susie (for multiple causal variants)

```r
library(susieR)

# Run SuSiE-based coloc for multiple signals
result <- coloc.susie(
  dataset1 = trait1,
  dataset2 = trait2
)
```

## Step 5: Result Interpretation

### Decision Rules

| PP4 Range | Interpretation | Action |
|-----------|----------------|--------|
| PP4 ≥ 0.8 | Strong evidence of colocalization | Prioritize gene |
| 0.5 ≤ PP4 < 0.8 | Moderate evidence | Consider with other evidence |
| PP4 < 0.5 | Weak/no evidence | Not colocalized |

### Additional Checks

1. **PP3 vs PP4**: High PP3 suggests different causal variants
2. **Power**: Check if sample size is adequate
3. **Sensitivity**: Run with different priors

### Sensitivity Analysis

```r
# Test with different priors
result_sensitive <- coloc.abf(
  dataset1 = trait1,
  dataset2 = trait2,
  p1 = 1e-4,  # Prior for trait 1
  p2 = 1e-4,  # Prior for trait 2
  p12 = 1e-5  # Prior for shared variant
)
```

## Step 6: Visualization

### Locus Compare Plot

```r
library(ggplot2)

# Create locus comparison plot
p <- ggplot() +
  geom_point(data = gwas_region, aes(x = POS, y = -log10(pval)), color = "blue") +
  geom_point(data = qtl_region, aes(x = POS, y = -log10(pval)), color = "red") +
  theme_minimal() +
  labs(
    title = paste0("Colocalization at ", gene),
    x = "Position",
    y = "-log10(P)"
  )

ggsave("locus_compare.pdf", p)
```

### Posterior Probability Bar Plot

```r
# Bar plot of posterior probabilities
pp_values <- c(result$summary[1], result$summary[2], result$summary[3], result$summary[4], result$summary[5])
names(pp_values) <- c("H0", "H1", "H2", "H3", "H4")

barplot(pp_values, 
  main = "Posterior Probabilities",
  ylab = "Probability",
  col = c("gray", "blue", "red", "purple", "green")
)
```

## Reporting Template

```markdown
## Colocalization Results

### Region Information
- Chromosome: chrX
- Region: start-end (±X kb around lead SNP/gene)
- Gene: GENE_NAME (for eQTL colocalization)

### Sample Information
- Trait 1 (GWAS): N = X, type = quantitative/case-control
- Trait 2 (QTL): N = X, type = quantitative

### Colocalization Results
| Hypothesis | Posterior Probability |
|------------|----------------------|
| H0 (no association) | X.XX |
| H1 (trait 1 only) | X.XX |
| H2 (trait 2 only) | X.XX |
| H3 (different variants) | X.XX |
| H4 (shared variant) | **X.XX** |

### Conclusion
- PP4 = X.XX → [Strong/Moderate/Weak] evidence of colocalization
- [Gene prioritized/not prioritized] for follow-up

### Sensitivity Analysis
[Results with different priors]
```

## Common Issues & Solutions

### Issue 1: Low power
- **Solution**: Ensure adequate sample size; consider larger region

### Issue 2: Multiple signals
- **Solution**: Use coloc.susie for conditional analysis

### Issue 3: Mismatched alleles
- **Solution**: Harmonize alleles before analysis

### Issue 4: LD mismatch
- **Solution**: Use LD reference from same population

## Alternative Methods

| Method | Description | When to Use |
|--------|-------------|-------------|
| coloc | Bayesian colocalization | Standard approach |
| eCAVIAR | Fine-mapping based | When fine-mapping needed |
| fastENLOC | Fast enrichment-based | Large-scale analysis |
| SMR/HEIDI | Single variant test | Single causal variant assumption |
| PAINTOR | Probabilistic annotation | With functional annotations |

## Reference Files

For detailed information, see:
- `references/coloc-hypotheses.md` - Detailed hypothesis descriptions
- `references/alternative-methods.md` - Other colocalization methods

## Dependencies

```r
# Required R packages
install.packages("coloc")
install.packages("susieR")
install.packages("ggplot2")
```

## Example Usage

**User**: "I want to test if the GWAS signal for BMI at the FTO locus colocalizes with eQTLs in adipose tissue"

**Response**:
1. Define region around FTO gene (chr16:53-54 Mb)
2. Extract BMI GWAS summary statistics
3. Extract adipose eQTL data for FTO
4. Run coloc analysis
5. Interpret PP4 value
6. Generate report and plots
