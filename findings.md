# Findings

This file summarises what the notebooks in this repository found. Each
section names the notebook the numbers come from.

## The data

The HCV dataset (`hcvdat0.csv`) has 615 patients and 12 predictors: age,
sex and ten laboratory values. Seven suspected blood donors were removed
because their true status is uncertain. This leaves 608 patients: 533
healthy, 24 with hepatitis, 21 with fibrosis and 30 with cirrhosis. Only
75 patients (12%) are sick, so the classes are very imbalanced.

## Cleaning (`cleaning.ipynb`)

- Five columns have missing values (31 values in total). ALP is missing
  mostly in sick patients, so rows with missing values were not deleted.
- Three ways of filling the gaps were compared. None was clearly better,
  so the median was used because it is the simplest.
- Outliers were reported but kept. All 30 cirrhosis patients have at least
  one outlier, so removing outliers would remove the disease itself.

## Exploration and statistics (`eda.ipynb`, `stats.ipynb`)

- None of the 11 numeric variables is normally distributed. CREA, BIL,
  GGT, AST, ALP and ALT are strongly right-skewed.
- 10 of the 11 variables differ between the four categories (Kruskal-Wallis
  and ANOVA agree). CREA is the only one that does not.
- AST, GGT and BIL separate the categories most strongly.

## Class imbalance (`class_imbalance.ipynb`)

Nine strategies were compared on four classes with logistic regression and
random forest, using 5-fold cross-validation repeated five times. All
resampling was done inside the training folds.

| Model | Strategy | Macro F1 | Balanced accuracy |
|---|---|---|---|
| Logistic regression | None | 0.653 | 0.625 |
| Logistic regression | Class weights | 0.602 | 0.667 |
| Logistic regression | Random under-sampling | 0.608 | 0.701 |
| Random forest | None | 0.590 | 0.566 |
| Random forest | SMOTE | 0.677 | 0.680 |
| Random forest | SMOTEENN | 0.682 | 0.689 |

- Random forest improved with SMOTE-type methods. Logistic regression did
  not improve on macro F1 with any strategy.
- Fibrosis recall stayed below 0.61 for every strategy. Resampling does not
  solve the hardest class.
- Models built for imbalance (Balanced Random Forest, EasyEnsemble,
  RUSBoost and others) did not beat random forest with SMOTE on macro F1.

## SMOTE leakage (`experiment_datasets.ipynb`)

This experiment shows what happens when SMOTE is applied to the whole
dataset before splitting it. Synthetic copies of the test patients then end
up in the training data, and the scores rise to as much as 99%.

| Task | Model | Macro F1, honest | Macro F1, leaky |
|---|---|---|---|
| Healthy vs sick | Logistic regression | 0.918 | 0.969 |
| Healthy vs sick | Random forest | 0.944 | 0.986 |
| Healthy vs sick | Decision tree | 0.881 | 0.963 |
| Four classes | Logistic regression | 0.620 | 0.876 |
| Four classes | Random forest | 0.659 | 0.994 |
| Four classes | Decision tree | 0.561 | 0.958 |

- Healthy vs sick is an easy task, so the leak adds little (0.04 to 0.08).
- With four classes the leak adds 0.26 to 0.40. The honest scores are
  between 0.56 and 0.66.
- A two-step model (healthy or not, then type of damage) was not clearly
  better than a single four-class model (macro F1 0.678 against 0.659).

## Machine learning models (`ml_models.ipynb`)

Three models were trained to predict healthy or sick, with the same
settings and evaluation as the COM725 study: an 80/20 stratified split,
class weights, and no synthetic patients. The AST/ALT ratio was added as an
engineered feature. The test set has 107 healthy and 15 sick patients.

| Model | Accuracy | Sick patients found | Healthy patients wrongly flagged |
|---|---|---|---|
| Logistic regression | 0.98 | 13 of 15 | 1 of 107 |
| Random forest | 0.98 | 13 of 15 | 1 of 107 |
| Decision tree | 0.98 | 15 of 15 | 2 of 107 |

All three models do well without any synthetic data.

## Feature engineering

- **COM725 data:** FIB-4 was added, using the formula of Sterling et al.
  (2006): (age x AST) / (platelets x square root of ALT). It is used by
  the logistic regression and random forest there, not by the two models
  trained on binned values. All four still perform at chance level (24%
  to 27% accuracy on four stages).
- **HCV data:** FIB-4 needs a platelet count, which this dataset does not
  have. The AST/ALT ratio (Sheth et al., 1998) was used instead.

## Limits

- The test set in `ml_models.ipynb` has only 15 sick patients and comes
  from a single split. One extra mistake changes sick recall by about 7
  points. The cross-validated macro F1 for the same task is 0.86 to 0.90.
- `ml_models.ipynb` uses the file imputed with the median of all patients,
  so 31 values were filled before the split.
- There are only 21 to 30 patients in each disease class and no external
  dataset to validate on.
- Model settings were fixed and not tuned.

## References

- Chicco, D. and Jurman, G. (2021) 'An ensemble learning approach for
  enhanced classification of patients with hepatitis and cirrhosis',
  *IEEE Access*, 9, pp. 24485-24498.
- Sheth, S.G., Flamm, S.L., Gordon, F.D. and Chopra, S. (1998) 'AST/ALT
  ratio predicts cirrhosis in patients with chronic hepatitis C virus
  infection', *American Journal of Gastroenterology*, 93(1), pp. 44-48.
- Sterling, R.K. et al. (2006) 'Development of a simple noninvasive index
  to predict significant fibrosis in patients with HIV/HCV coinfection',
  *Hepatology*, 43(6), pp. 1317-1325.
- Vallet-Pichard, A. et al. (2007) 'FIB-4: an inexpensive and accurate
  marker of fibrosis in HCV infection. Comparison with liver biopsy and
  FibroTest', *Hepatology*, 46(1), pp. 32-36.
- Wei, R. et al. (2018) 'Clinical prediction of HBV and HCV related
  hepatic fibrosis using machine learning', *EBioMedicine*, 35,
  pp. 124-132.
