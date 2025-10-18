# Leakage-Safe Breast Cancer Risk Classification with TabTransformer

This repository contains the full implementation and results of my Georgetown University M.S. Biostatistics practicum project.  
The objective of this work is to develop a **leakage-safe machine learning pipeline** that predicts **breast cancer risk levels (High vs. Average)** using both gene expression data and clinical categorical variables.

Key highlights:
- Built a **cross-validated, leakage-safe** feature selection pipeline.
- Compared **TabTransformer**, **Random Forest**, **LASSO**, and **Logistic Regression** models.
- Applied the pipeline to RNA-seq dataset **GSE164641** after normalization and preprocessing.

This project serves both as a practicum deliverable and as a reproducible benchmark for evaluating hybrid tabular models that combine biological and clinical data.

## Methodology

### Data Sources and Preprocessing
- **Dataset:** GSE164641 RNA-seq gene expression and patient metadata.
- **Continuous features:** Normalized using **DESeq2** in Python to obtain variance-stabilized counts.
- **Gene selection threshold:** `padj < 0.05` and `|log2FC| > 1`.
- **Categorical features:**
  - Converted numeric variables (e.g., BMI, Age, Age of Menarche) into standard bins.
  - Performed **Chi-square tests** on categorical features within each training fold to select significant ones (`p < 0.05`).
  - Removed `tyrer_cuzick_score` to prevent label leakage.

### Model Setup
| Model | Type | Description |
|--------|------|-------------|
| **TabTransformer** | Deep learning | Transformer-based model for tabular data implemented via [lucidrains/tab-transformer-pytorch](https://github.com/lucidrains/tab-transformer-pytorch) |
| **Random Forest** | Tree-based ensemble | Nonlinear classifier using multiple decision trees |
| **LASSO Regression** | Linear | L1-regularized logistic regression for feature sparsity |
| **Logistic Regression** | Classical baseline | Traditional generalized linear model |

### Cross-Validation Strategy
- Used **5-fold stratified cross-validation**.
- Each fold followed these steps:
  1. Split 80% training and 20% validation.
  2. Perform feature selection *only on the training subset* to avoid leakage.
  3. Train and evaluate models on the validation subset.
- Model performance metrics (ROC-AUC, precision, recall, F1) were averaged across folds.

### Evaluation Metrics
- **ROC-AUC:** Primary performance measure, averaged with standard deviation.
- **Classification reports:** Macro- and weighted-average scores aggregated across folds.
- **Visualization:** Three summary plots showing
  1. Continuous-only model comparison  
  2. TabTransformer internal comparison (cont / cat / combined)  
  3. Combined-feature model comparison

## Results and Outputs

All model outputs, evaluation metrics, and figures are automatically saved in the `/output` directory after running the notebook.

### Summary Tables
| File | Description |
|------|-------------|
| **cv_auc_summary.csv** | Mean and standard deviation of AUC scores across 5 folds |
| **cv_classification_report_macro.csv** | Averaged macro precision, recall, and F1-scores |
| **cv_classification_report_weighted.csv** | Averaged weighted precision, recall, and F1-scores |

### Visualization Plots
| File | Description |
|------|-------------|
| **cv_roc_only_continuous.png** | ROC-AUC comparison among all models using *only continuous* (gene) features |
| **cv_roc_tt_internal.png** | TabTransformer internal comparison: continuous-only, categorical-only, and combined models |
| **cv_roc_combined.png** | ROC-AUC comparison among all models using *combined* features |

These figures display the **mean ROC curves** with shaded regions representing the **±1 standard deviation** interval from 5-fold cross-validation.

---

### Example of Model Performance

| Model | Feature Type | Mean AUC | Std. Dev. |
|--------|---------------|-----------|-----------|
| TabTransformer | Combined | 0.91 | ±0.03 |
| Random Forest | Combined | 0.88 | ±0.04 |
| LASSO Regression | Continuous | 0.85 | ±0.05 |
| Logistic Regression | Continuous | 0.83 | ±0.06 |

> *Values are illustrative; see `cv_auc_summary.csv` for exact numerical results.*

---

### Classification Report Interpretation
- **Macro average:** Treats all classes equally, regardless of sample count.  
- **Weighted average:** Adjusts scores according to class frequencies.  
- **Recommendation:** Macro is best for overall model fairness; weighted reflects real-world imbalanced performance.

---

### Additional Outputs
- The pipeline prints model summaries (`print(model)`) to display architecture and parameters for TabTransformer and other models.  
- Fold-level intermediate results can be inspected in the notebook’s execution cells.
