# GWAS Databases Reference

## IEU GWAS Database

The largest curated GWAS database, accessible via TwoSampleMR.

### Access

```r
library(TwoSampleMR)

# List available GWAS
ao <- available_outcomes()

# Search by trait
ao[grepl("body mass", ao$trait, ignore.case = TRUE), ]
```

### Common GWAS IDs

| Trait | ID | Population | Sample Size | Year |
|-------|-----|------------|-------------|------|
| BMI | ieu-a-2 | European | 322,154 | 2015 |
| Height | ieu-a-812 | European | 252,901 | 2015 |
| WHR | ieu-a-73 | European | 115,588 | 2015 |
| T2D | ebi-a-GCST006867 | European | 74,124 | 2017 |
| CAD | ieu-a-7 | European | 184,305 | 2015 |
| Stroke | ebi-a-GCST006903 | European | 438,847 | 2018 |
| Alzheimer's | ieu-a-237 | European | 63,826 | 2013 |
| Parkinson's | ieu-a-298 | European | 13,706 | 2014 |
| Schizophrenia | ebi-a-GCST006728 | European | 36,989 | 2017 |
| Depression | ebi-a-GCST006614 | European | 135,458 | 2018 |
| Education | ieu-a-301 | European | 328,928 | 2016 |
| IQ | ebi-a-GCST006990 | European | 77,545 | 2018 |

---

## OpenGWAS

Alternative database with more recent GWAS.

### Access

```r
# Use OpenGWAS backend
ieugwasr::gwasinfo()
```

---

## GWAS Catalog

Manually curated GWAS from literature.

### URL
https://www.ebi.ac.uk/gwas/

### Download

```bash
# Download full catalog
wget https://www.ebi.ac.uk/gwas/api/search/downloads/alternative
```

---

## FinnGen

Finnish biobank GWAS.

### URL
https://www.finngen.fi/

### Access

```r
# FinnGen data via TwoSampleMR
outcome <- extract_outcome_data(
  snps = exposure$SNP,
  outcomes = "finn-a-T2D"
)
```

---

## UK Biobank

Large-scale UK biobank GWAS.

### Sources
- Neale Lab: http://www.nealelab.is/uk-biobank
- Pan-UKBB: https://pan.ukbb.broadinstitute.org/
- OpenGWAS: Various UKBB traits available

---

## eQTL Databases

For exposure data from eQTL studies:

| Database | Tissue | Access |
|----------|--------|--------|
| GTEx v8 | 49 tissues | https://gtexportal.org/ |
| eQTLGen | Blood | https://eqtlgen.org/ |
| PsychENCODE | Brain | http://psychencode.org/ |

---

## Format Requirements

### Standard Format

```
SNP  beta  se  pval  eaf  effect_allele  other_allele
rs123  0.05  0.01  1e-10  0.45  A  G
```

### Column Mapping

| Column | Description | Required |
|--------|-------------|----------|
| SNP | rsID or chr:pos | Yes |
| beta | Effect size | Yes |
| se | Standard error | Yes |
| pval | P-value | Yes |
| eaf | Effect allele frequency | Recommended |
| effect_allele | Effect allele | Yes |
| other_allele | Non-effect allele | Yes |
