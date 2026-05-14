# MR Methods Reference

## Inverse Variance Weighted (IVW)

**Assumptions**:
- No horizontal pleiotropy (all SNPs affect outcome only through exposure)
- All instruments valid (no direct effect on outcome)

**Formula**:
$$\beta_{IVW} = \frac{\sum_j w_j \beta_{Yj} / \beta_{Xj}}{\sum_j w_j}$$

where $w_j = \sigma_{Yj}^{-2}$

**When to use**: Primary method, most statistical power

---

## MR-Egger

**Assumptions**:
- InSIDE assumption (instrument strength independent of direct effect)
- Linear relationship between instrument-exposure and instrument-outcome effects

**Formula**:
$$E[\beta_{Yj}] = \alpha + \beta \beta_{Xj}$$

where $\alpha$ is the intercept (detects directional pleiotropy)

**Intercept interpretation**:
- $\alpha = 0$: No directional pleiotropy
- $\alpha \neq 0$: Directional pleiotropy present

**When to use**: Detect and correct for directional pleiotropy

---

## Weighted Median

**Assumptions**:
- At least 50% of instruments are valid

**Formula**:
$$\beta_{WME} = \text{median}(w_j, \beta_{Yj}/\beta_{Xj})$$

**When to use**: Robust to outliers, valid when majority of instruments are valid

---

## Weighted Mode

**Assumptions**:
- Largest subset of instruments share the same causal estimate

**When to use**: Very robust, valid when modal estimate is correct

---

## MR-PRESSO

**Components**:
1. Global test: Detect overall horizontal pleiotropy
2. Outlier test: Identify outlier SNPs
3. Distortion test: Compare estimates before/after outlier removal

**When to use**: Detect and correct for outlier-driven pleiotropy

---

## Sensitivity Metrics

### Heterogeneity (Cochran's Q)

$$Q = \sum_j w_j (\beta_{Yj}/\beta_{Xj} - \beta_{IVW})^2$$

- High Q → heterogeneous estimates → potential invalid instruments

### MR-Egger Intercept

- Tests for directional pleiotropy
- Significant p-value → pleiotropy present

### Leave-One-Out

- Remove each SNP and recalculate estimate
- Large changes → influential outlier
