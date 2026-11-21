# hcv-ML-DS-project
Exploratory analysis and machine learning evaluation of hepatitis C clinical data for liver fibrosis staging, using Python, statistical testing, classification models and a Power BI dashboard.

## How to reproduce

All analyses run on Python 3.10 with the package versions in `requirements.txt`:

```
python3.10 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Run the notebooks from the project folder in this order. `cleaning.ipynb` writes `hcv_median_imputed.csv`, which `eda.ipynb` and `stats.ipynb` read, so it must run first. The other notebooks read the raw `hcvdat0.csv` and do their own cleaning inside each training fold.

1. `cleaning.ipynb` - missing values, imputation comparison, outliers, missingness by sex
2. `eda.ipynb` - descriptive tables and plots, lab values by sex
3. `stats.ipynb` - normality, Kruskal-Wallis, ANOVA, differences by sex
4. `class_imbalance.ipynb` - comparison of nine class imbalance strategies
5. `ml_models.ipynb` - model development and internal validation
6. `data_leakage.ipynb` - re-analysis of Das et al. (2025)

`COM725/COM725_AE2_Omar_Al_Hajaj.ipynb` holds the earlier analysis of the Egyptian HCV dataset.

Data: UCI Machine Learning Repository, HCV data (Lichtinghagen, Klawonn and Hoffmann, 2020), doi:10.24432/C5D612.
