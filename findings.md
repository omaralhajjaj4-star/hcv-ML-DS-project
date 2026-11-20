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

## Data leakage (`data_leakage.ipynb`)

This notebook repeats the pipeline of Das et al. (2025), who applied mean
imputation, winsorising, SMOTE and scaling to all 615 rows and only then
split the data 70/15/15. Their test set has 400 rows, 81 of them suspect
blood donors, although the raw data hold only 7. The notebook repeats the
pipeline with four models at default settings, then splits first and fits
every step on the training rows only. The paper does not state its seed,
scaler or winsorising limits, so 5% and 95% limits and `random_state=42`
were assumed and the replication only approximates the paper.

- Replicated: the test set has 400 rows with 80 in each class, including
  80 suspect blood donor rows, although only 7 exist.
- Corrected: the test set has 93 real patients: 81 blood donors, 1 suspect
  donor, 3 hepatitis, 3 fibrosis and 5 cirrhosis.

| Model | Paper accuracy | Replicated accuracy | Corrected accuracy | Replicated macro F1 | Corrected macro F1 |
|---|---|---|---|---|---|
| Logistic regression | 92.50 | 91.25 | 93.55 | 0.911 | 0.603 |
| Decision tree | 94.75 | 96.75 | 93.55 | 0.968 | 0.684 |
| Random forest | 98.25 | 99.75 | 95.70 | 0.997 | 0.759 |
| SVM | 98.50 | 99.75 | 95.70 | 0.997 | 0.659 |

- The replication gets close to the paper's accuracy (within 2.5 points).
- Corrected accuracy stays at 93.55 to 95.70 because 81 of the 93 test
  patients are blood donors. Macro F1 falls from 0.911 to 0.997 down to 0.603
  to 0.759, which shows the effect of the order of steps.

## Machine learning models (`ml_models.ipynb`)

Three models (logistic regression, random forest and decision tree) were
trained in three ways, with the same settings and evaluation as the COM725
study: an 80/20 stratified split and a median imputer fitted inside each
pipeline, so the training rows alone set the median.

**Healthy or sick** (no SMOTE). The test set has 107 healthy and 15 sick
patients.

| Model | Accuracy | Macro F1 | Sick patients found |
|---|---|---|---|
| Logistic regression | 0.98 | 0.94 | 13 of 15 |
| Random forest | 0.98 | 0.94 | 13 of 15 |
| Decision tree | 0.98 | 0.96 | 15 of 15 |

**Four classes** (SMOTE inside the pipeline). The test set has 107 healthy,
6 cirrhosis, 5 hepatitis and 4 fibrosis patients.

| Model | Accuracy | Macro F1 | Cirrhosis | Hepatitis | Fibrosis |
|---|---|---|---|---|---|
| Logistic regression | 0.92 | 0.61 | 4 of 6 | 3 of 5 | 1 of 4 |
| Random forest | 0.93 | 0.58 | 6 of 6 | 1 of 5 | 1 of 4 |
| Decision tree | 0.92 | 0.47 | 5 of 6 | 1 of 5 | 0 of 4 |

**Three stages** (sick patients only). The test set has 6 cirrhosis,
5 hepatitis and 4 fibrosis patients.

| Model | Accuracy | Macro F1 | Cirrhosis | Hepatitis | Fibrosis |
|---|---|---|---|---|---|
| Logistic regression | 0.67 | 0.65 | 5 of 6 | 3 of 5 | 2 of 4 |
| Random forest | 0.60 | 0.55 | 5 of 6 | 3 of 5 | 1 of 4 |
| Decision tree | 0.60 | 0.53 | 6 of 6 | 1 of 5 | 2 of 4 |

- Healthy or sick is easy: all three models reach 0.98 accuracy without
  synthetic patients.
- Naming the disease is much harder. Four-class accuracy stays at 0.92 to
  0.93 only because most patients are healthy, and macro F1 falls to 0.47
  to 0.61.
- Cirrhosis is found most often. Fibrosis and hepatitis are often mixed up.

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
  points.
- The disease-class test sets are small: 4 to 6 patients per class in
  `ml_models.ipynb` and 1 to 5 in `data_leakage.ipynb`, so one patient
  changes a recall by about 17 points or more.
- There are only 21 to 30 patients in each disease class and no external
  dataset to validate on.
- Model settings were fixed and not tuned.

## References

- Chicco, D. and Jurman, G. (2021) 'An ensemble learning approach for
  enhanced classification of patients with hepatitis and cirrhosis',
  *IEEE Access*, 9, pp. 24485-24498.
- Das, N. et al. (2025) 'Enlightened prognosis: Hepatitis prediction with
  an explainable machine learning approach', *PLoS ONE*, 20(3), e0319078.
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
