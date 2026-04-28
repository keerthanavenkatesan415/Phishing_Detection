# Phishing URL Detection: A Hybrid Clustering + Supervised Learning Approach

## Problem Statement

Phishing URLs represent a significant cybersecurity threat. This project addresses binary classification of URLs as legitimate or phishing using a hybrid pipeline that combines unsupervised clustering (feature extraction) with supervised classification models. The goal is to identify malicious URLs with high precision and recall while maintaining interpretability through feature importance and SHAP analysis.

## Dataset

- **Source**: UCI Machine Learning Repository (Phishing URL dataset)
- **Samples**: 11,055 URLs total
- **Features**: 30 binary/ternary features encoding URL characteristics
- **Classes**: 
  - Legitimate URLs (label: -1)
  - Phishing URLs (label: 1)
- **Train/Test Split**: 80/20 stratified (8,844 train / 2,211 test)
- **Missing Values**: 0 (complete dataset)

## Feature Engineering

### Raw Features (30 total)
URL morphology, domain properties, server-side characteristics, and page-based features including:
- `SSLfinal_State`: SSL certificate validity status
- `URL_of_Anchor`: Anchor tag legitimacy
- `Prefix_Suffix`: URL prefix/suffix patterns
- `web_traffic`: Presence in traffic datasets
- And 26 additional features (see EDA output for full list)

### Clustering Features
Two unsupervised clustering algorithms generate additional features:
1. **K-Means Clustering**: Silhouette score optimization (Range: k=2 to 11)
2. **DBSCAN Clustering**: Parameter tuning via grid search (eps=3.0, min_samples=10)

These cluster assignments augment the feature space for supervised learning.

## Project Structure

```
Phishing_Detection/
├── README.md                                    # This file
├── requirements.txt                             # Python package dependencies
├── environment.yml                              # Conda environment specification
├── preprocessing&clustering.ipynb               # Phase 1-4: Data prep & clustering
├── 02_modeling_and_eval.ipynb                   # Phase 5-6: Classification & interpretation
└── data/
    ├── dataset.arff                             # Raw dataset
    └── processed/
        ├── X_train.csv, X_test.csv              # Raw features (unscaled)
        ├── X_train_scaled.csv, X_test_scaled.csv # Normalized features
        ├── y_train.csv, y_test.csv              # Labels
        ├── kmeans_train.npy, kmeans_test.npy    # K-Means cluster assignments
        └── dbscan_train.npy, dbscan_test.npy    # DBSCAN cluster assignments
```

## Methodology

### Phase 1-2: Data Loading & Preprocessing
- Load ARFF dataset and convert to DataFrame
- Verify zero missing values and feature distributions
- Stratified train/test split (80/20)
- StandardScaler normalization (fit on train, transform on test)
- Data leakage verification

### Phase 3: Unsupervised Clustering
**K-Means**:
- Silhouette score optimization to find optimal k
- Generate cluster assignments for both train and test sets

**DBSCAN**:
- Parameter grid search for eps and min_samples
- Generate density-based cluster assignments

### Phase 4: Cluster Interpretation
- Per-cluster phishing rate analysis
- Feature mean values by cluster
- PCA visualization
- Cluster size and density analysis

### Phase 5: Supervised Classification
Three classification algorithms with increasing complexity:

1. **Logistic Regression** (baseline, no clusters)
   - Max iterations: 1000
   - Trained on: X_train (base features only)

2. **Random Forest** (tuned with clusters)
   - GridSearchCV with 3-fold CV, F1 scoring
   - Parameter grid: n_estimators [100, 200], max_depth [None, 10]
   - Trained on: X_train_both (base + cluster features)
   - **Result**: Best model (F1: 0.976)

3. **XGBoost** (tuned with clusters)
   - GridSearchCV with 3-fold CV, F1 scoring
   - Parameter grid: n_estimators [100, 200, 300], max_depth [3, 4, 5], learning_rate [0.01, 0.05, 0.1]
   - Trained on: X_train_both (base + cluster features)
   - **Result**: F1: 0.972

### Phase 6: Interpretability & Analysis
- **SHAP TreeExplainer**: Global feature importance via summary plots
- **Permutation Importance**: Cross-model feature ranking
- **Feature Consensus**: Identify universally important features
- **Ablation Studies**: Quantify clustering contribution
- **Cross-Validation**: 5-fold CV with uncertainty quantification
- **Error Analysis**: Confusion matrices, false positive/negative rates

## Results

### Model Performance

| Model | Accuracy | Precision | Recall | F1-Score | CV F1 (mean±std) |
|-------|----------|-----------|--------|----------|-----------------|
| Logistic Regression | 92.90% | 92.68% | 92.67% | 0.9288 | 0.9266±0.0052 |
| Random Forest | **97.60%** | **97.00%** | **96.99%** | **0.9760** | **0.9699±0.0038** |
| XGBoost | 97.24% | 96.70% | 96.70% | 0.9724 | 0.9670±0.0035 |

### Key Findings

**1. Clustering Impact (Ablation Study)**
- Random Forest baseline (no clusters): F1 = 0.9504
- Random Forest with clusters: F1 = 0.9760 (**+2.56%** improvement)
- XGBoost baseline (no clusters): F1 = 0.9487
- XGBoost with clusters: F1 = 0.9724 (**+2.50%** improvement)

**Conclusion**: Clustering features provide measurable, consistent improvement in supervised classification performance.

**2. Feature Importance Consensus**
Across all three models, top-3 consensus features:
- `SSLfinal_State` (SSL certificate validity)
- `URL_of_Anchor` (anchor tag characteristics)
- `Prefix_Suffix` (URL prefix/suffix patterns)

**3. Error Characteristics**
- Random Forest error rate: 2.40% (53 total errors)
  - False positive rate: 3.47% (legitimate as phishing)
  - False negative rate: 1.54% (phishing missed)
- Cross-model agreement: 91.59% of test samples classified consistently

## How to Reproduce

### Setup (Option 1: pip)
```bash
# Clone repository
cd Phishing_Detection

# Install dependencies
pip install -r requirements.txt

# (macOS only) Install OpenMP for XGBoost
brew install libomp
```

### Setup (Option 2: conda)
```bash
# Create environment
conda env create -f environment.yml

# Activate environment
conda activate phishing-detection
```

### Run Analysis
```bash
# Start Jupyter
jupyter notebook

# Execute notebooks in order:
1. preprocessing&clustering.ipynb       # Phases 1-4 (Data + Clustering)
2. 02_modeling_and_eval.ipynb          # Phases 5-6 (Classification + Interpretation)
```

## Execution Order & Cell Descriptions

### `preprocessing&clustering.ipynb`
- **Cells 1-3**: Data loading, exploration, train/test split
- **Cells 4-8**: K-Means clustering with silhouette optimization
- **Cells 9-12**: DBSCAN clustering with parameter tuning
- **Cells 13-28**: Cluster interpretation (phishing rates, visualizations)

### `02_modeling_and_eval.ipynb`
- **Cells 1-2**: Data loading and preparation
- **Cells 3-8**: Baseline and variant Random Forest models
- **Cell 9**: Random Forest GridSearchCV optimization
- **Cell 10**: XGBoost GridSearchCV optimization (with cluster features)
- **Cell 11**: Model evaluation and SHAP analysis
- **Cell 12**: 5-Fold cross-validation results
- **Cell 13**: Detailed error analysis (confusion matrices)
- **Cell 14**: Ablation studies (clustering impact)
- **Cell 15**: Feature importance comparison

## Dependencies

See `requirements.txt` or `environment.yml` for complete list.

Primary packages:
- `pandas` (data manipulation)
- `scikit-learn` (ML algorithms, preprocessing)
- `xgboost` (gradient boosting)
- `shap` (feature importance interpretation)
- `matplotlib`, `seaborn` (visualization)

## Key Design Decisions

1. **Hybrid Approach**: Combining unsupervised clustering with supervised learning leverages structural patterns in the feature space while maintaining discriminative power.

2. **Feature Consistency**: All models trained on same feature sets (base features or base + clusters) for fair comparison.

3. **Rigorous Tuning**: Both RF and XGBoost subject to GridSearchCV with F1 scoring and cross-validation, ensuring equal methodological rigor.

4. **Interpretability**: SHAP + permutation importance + ablation studies provide multi-faceted understanding of model decisions.

5. **Stratified Split**: Maintains class distribution across train/test, preventing bias.

## Reproducibility Notes

- All random seeds fixed (random_state=42)
- Cross-validation folds deterministic
- Results may vary slightly due to XGBoost's stochastic tree construction (deterministic within 0.001 F1 variation)
- OpenMP library required on macOS for XGBoost performance

## Future Improvements

1. Implement stratified k-fold cross-validation for uncertainty estimates
2. Expand SHAP analysis (force plots, waterfall plots, dependence plots)
3. Explore advanced feature engineering (interaction terms, domain knowledge features)
4. Investigate ensemble methods (voting, stacking)
5. Perform statistical significance testing across models
6. Extend to URL phishing detection (beyond binary classification)

## References

- Dataset: [UCI ML Repository - Phishing URL Dataset](https://archive.ics.uci.edu/ml/datasets/Phishing+Websites)
- SHAP: [Lundberg & Lee (2017) - A Unified Approach to Interpreting Model Predictions](https://arxiv.org/abs/1705.07874)
- XGBoost: [Chen & Guestrin (2016) - XGBoost: A Scalable Tree Boosting System](https://arxiv.org/abs/1603.02754)

---

**Author**: [Your Name]  
**Date**: April 2026  
**Course**: CS439 (Data Mining & Machine Learning)
