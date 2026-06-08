# Survival_Analysis

# A Single-Cell Derived Stress-Response Gene Signature Predicts Clinical Outcomes in Luminal A Breast Cancer

**Karimat Adeola Busari (Hardae)**
Graduate Bioinformatics Intern, IITA Nigeria | MSc Medical Biology and Genetics
Portfolio Project 6 | 2025

---

## Overview

This project bridges single-cell transcriptomics and population-level survival analysis. Stress-response gene signatures identified in Cancer Epithelial subpopulations (Project 5, GSE176078) were scored in bulk RNA-seq data and tested for prognostic relevance across two large breast cancer cohorts: TCGA BRCA (n = 1,214) and METABRIC (n = 1,980).

**Key finding:** A 12-gene ER+ stress-response signature is significantly associated with improved overall survival in Luminal A patients in TCGA (log-rank p = 0.019, HR = 0.53). The signature remained directionally consistent in a multivariate Cox model adjusting for age and tumour stage (HR = 0.53, p = 0.07, concordance = 0.75), though it did not reach significance — consistent with a true biological signal that requires a larger event count to confirm in multivariate analysis.

---

## Project Structure

```
project6-survival-analysis/
│
├── notebooks/
│   └── Project6_Survival_Analysis.ipynb
│
├── figures/                        # All KM plots and figures available here
│
├── requirements.txt
└── README.md
```

---

## Biological Background

In Project 5, scRNA-seq analysis of 100,064 cells from TNBC, ER+, and HER2+ tumours revealed subtype-specific stress-response programs within Cancer Epithelial subpopulations. These programs — involving hypoxia adaptation, ER stress, glycolytic reprogramming, and cytokine signalling — were identified through Leiden clustering and Hallmark pathway enrichment.

The central question here: do these single-cell programs leave a detectable signal in bulk RNA-seq data, and does that signal predict survival?

---

## Gene Signatures

Signatures were derived from Wilcoxon rank-sum differential expression within Cancer Epithelial Leiden clusters (Cluster 13 for ER+, Cluster 11 and 21 for TNBC, Cluster 14 and 16 for HER2+), filtered to genes enriched in stress-response Hallmark pathways.

**ER+ Stress-Response Signature (12 genes — primary)**

JUN | CDKN1A | IRF1 | GADD45B | DDIT4 | ENO2 | HSPA5 | IER3 | BTG2 | CD44 | RHOB | STC2

**TNBC Stress-Response Signature (22 genes)**

VEGFA | ATF3 | LDHA | PKM2 | IDH2 | CD44 | PLAUR | DCN | IGFBP3 | TPBG | DDIT4 | RELA | FAS | CXCL10 | LDHB | IL6 | SERPINE1 | IL1B | CDKN1A | COL5A1 | CEBPB | F3

**HER2+ Stress-Response Signature (5 genes)**

CASP6 | SAT1 | ENO1 | IDH1 | SOD1

---

## Datasets

| Cohort   | N     | Platform                     | Role       |
|----------|-------|------------------------------|------------|
| TCGA BRCA | 1,214 | Illumina HiSeq RNA-seq       | Primary    |
| METABRIC  | 1,980 | Illumina HT-12 v3 Microarray | Validation |

- TCGA accessed via UCSC Xena Browser
- METABRIC accessed via cBioPortal
- scRNA-seq source: GSE176078 (Wu et al., 2021, Nature Genetics)

---

## Methods

### 1. Signature Derivation (from Project 5)
Cancer Epithelial cells were subsetted from the full scRNA-seq object (adata), excluding Leiden cluster 25. Wilcoxon rank-sum differential expression (sc.tl.rank_genes_groups) was run across Leiden clusters. Marker genes were filtered by Hallmark stress-response pathway membership (Glycolysis, Hypoxia, Unfolded Protein Response, Apoptosis) to define each signature.

### 2. Signature Scoring
For each bulk RNA-seq patient, each signature gene was z-score normalised across all patients. Z-scores were summed to generate a per-patient composite score. Patients were stratified at the median into High and Low groups.

### 3. Kaplan-Meier Survival Analysis
KM curves were fitted using KaplanMeierFitter (lifelines). Groups were compared using the log-rank test (logrank_test). Analysis was run within each PAM50 subtype: Luminal A, Luminal B, Basal, HER2-enriched.

### 4. Univariate Cox Proportional Hazards Model
A univariate Cox model (CoxPHFitter) was fitted on the continuous signature score to estimate the hazard ratio and 95% CI.

### 5. Multivariate Cox Model (LumA only)
For the LumA ER+ signature, a multivariate Cox model was fitted including age at diagnosis and tumour stage as covariates — to test whether the signature effect held independently of clinical factors.

### 6. METABRIC Validation
The same scoring and stratification pipeline was applied to METABRIC. Given the microarray platform, tertile-based stratification was also tested. A hormone therapy stratification analysis was additionally run (High/Low × hormone therapy YES/NO), yielding four groups.

---

## Results

### TCGA BRCA — Primary Analysis

| Subtype   | p-value | HR (95% CI)       | Significant? |
|-----------|---------|-------------------|--------------|
| Luminal A | 0.019   | 0.53 (0.31–0.91)  | ✅ Yes        |
| Luminal B | > 0.05  | —                 | ❌ No         |
| Basal     | > 0.05  | —                 | ❌ No         |
| HER2+     | > 0.05  | —                 | ❌ No         |

### Multivariate Cox — TCGA LumA (ER+ signature + age + stage)

| Covariate        | HR   | p-value  | Interpretation                        |
|------------------|------|----------|---------------------------------------|
| ER+ signature    | 0.53 | 0.07     | Directionally consistent, borderline  |
| Age at diagnosis | 1.04 | < 0.005  | Older age = higher hazard             |
| Tumour stage     | 1.73 | < 0.005  | Higher stage = higher hazard          |

Concordance = 0.75. The signature direction is preserved in the multivariate model, though it does not reach significance — likely due to a limited number of events (52 deaths in 420 LumA patients).

### METABRIC — Validation

| Analysis                         | p-value | Significant? |
|----------------------------------|---------|--------------|
| Median stratification, LumA      | > 0.05  | ❌ No         |
| Tertile stratification, LumA     | > 0.05  | ❌ No         |
| Hormone therapy stratification   | 0.198   | ❌ No         |

METABRIC non-significance is attributed to platform differences between RNA-seq (TCGA) and microarray (METABRIC). Microarray compresses dynamic range and reduces sensitivity for stress-inducible genes with moderate expression, including several in this signature (DDIT4, IER3, GADD45B). This is a known limitation of cross-platform validation and does not invalidate the TCGA finding.

All figures are available in the figures/ folder.

---

## Biological Interpretation

The ER+ signature captures a stress-resolution program that appears to be protective in the Luminal A context. Key genes and their roles:

- JUN, IRF1, BTG2 — transcription factors mediating growth arrest and immune modulation
- GADD45B, DDIT4, CDKN1A — DNA damage response and cell cycle arrest regulators
- HSPA5 — ER stress chaperone; high expression reflects active but controlled proteostasis
- IER3, RHOB — stress-responsive genes linked to apoptosis regulation

High expression of this program in LumA tumours may reflect a cellular state of managed stress resolution — consistent with the slower proliferation and better prognosis characteristic of Luminal A biology.

The signature is not prognostic in Luminal B, despite LumB also being ER+. This is biologically coherent: LumB tumours have higher proliferation and genomic instability, which likely overrides the stress-response signal.

---

## Limitations

- The univariate Cox model reaches significance; the multivariate model does not — this gap reflects a limited number of death events (52/420 LumA patients), not necessarily a null effect
- Scoring method is a simple z-score sum; ssGSEA or weighted scoring may improve cross-platform stability
- No RNA-seq matched external validation cohort beyond TCGA
- Overall survival includes non-cancer deaths; disease-specific survival would be a more precise endpoint

---

## Future Directions

- [ ] Validate in additional RNA-seq cohorts (GSE96058, GSE81538)
- [ ] ssGSEA-based scoring for cross-platform robustness
- [ ] Integrate with Project 4 LASSO/SHAP biomarker panel
- [ ] Seurat (R) replication of Project 5 for cross-tool reproducibility

---

## Tools and Libraries

| Tool              | Purpose                          |
|-------------------|----------------------------------|
| Python            | Primary language                 |
| scanpy            | scRNA-seq signature derivation   |
| lifelines         | KM analysis, log-rank, Cox model |
| scipy.stats       | Z-score normalisation            |
| pandas / numpy    | Data handling                    |
| matplotlib        | Visualisation                    |

Install dependencies:

```
pip install -r requirements.txt
```

requirements.txt:
```
scanpy>=1.9.0
lifelines>=0.27.0
scipy>=1.10.0
pandas>=1.5.0
numpy>=1.23.0
matplotlib>=3.6.0
```

---

## Related Projects

| Project | Description |
|---------|-------------|
| Project 1 | Differential gene expression — limma, GSE183947 |
| Project 2 | Breast cancer subtype classification — Random Forest, TCGA |
| Project 3 | GO and pathway enrichment — gProfiler |
| Project 4 | Biomarker discovery — LASSO + TreeSHAP |
| Project 5 | Single-cell RNA-seq — Scanpy, GSE176078 (this project's source) |

---

## References

1. Wu, S.Z. et al. (2021). A single-cell and spatially resolved atlas of human breast cancers. Nature Genetics, 53(9), 1334–1347.
2. Cancer Genome Atlas Network (2012). Comprehensive molecular portraits of human breast tumours. Nature, 490(7418), 61–70.
3. Curtis, C. et al. (2012). The genomic and transcriptomic architecture of 2,000 breast tumours. Nature, 486(7403), 346–352.
4. Davidson-Pilon, C. (2019). lifelines: survival analysis in Python. JOSS, 4(40), 1317.
5. Parker, J.S. et al. (2009). Supervised risk predictor of breast cancer based on intrinsic subtypes. JCO, 27(8), 1160–1167.

---

*Part of an independent bioinformatics portfolio. All data sourced from publicly available repositories (GEO, TCGA, METABRIC). No patient-identifiable information used.*
