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
- Dunn's post-hoc test: hepatitis and fibrosis differ only in age. No lab
  value separates these two stages, which is why every model confuses them.
- Women and men differ in 7 of the 11 measurements in all patients and in
  8 of 11 among healthy donors (higher liver enzymes and creatinine in
  men), so sex is kept as a predictor and results are reported by sex.
- Three healthy donors have albumin equal to their ALP, one with albumin
  above total protein. These are most likely entry errors. They are kept,
  and a sensitivity analysis shows they change nothing.

## Sample size (`stats.ipynb`)

- Healthy vs sick needs between 398 and 1,417 patients (50 to 175 sick),
  depending on how well the predictors are expected to work (Riley et al.,
  2020; 2021). 608 patients with 75 sick is borderline.
- Four classes would need about 13,000 patients and three stages about
  1,600 (Pate et al., 2023). The data are far too small, so all
  multiclass results are exploratory.

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

Over 50 seeds (`data_leakage.ipynb`, "Repeating both versions over 50
seeds"), the leakage effect holds every time:

| Model | Leaky macro F1 (median) | Corrected macro F1, 5 classes (median, 2.5-97.5%) | Corrected macro F1, 4 real classes (median) | Paired drop (median) |
|---|---|---|---|---|
| Logistic regression | 0.905 | 0.541 (0.354-0.735) | 0.609 | 0.362 |
| Decision tree | 0.964 | 0.450 (0.310-0.689) | 0.527 | 0.499 |
| Random forest | 0.993 | 0.558 (0.376-0.790) | 0.611 | 0.432 |
| SVM | 0.992 | 0.536 (0.372-0.755) | 0.619 | 0.460 |

- The drop is positive in all 50 seeds for all four models.
- Every corrected test set holds exactly one suspect blood donor, so that
  class scores 0 or 1 and makes up a fifth of macro F1. Macro F1 over the
  four real classes is the fairer corrected measure.

The single split with seed 42 (table below) was a favourable split:

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
  patients are blood donors. At seed 42, macro F1 falls from 0.911 to 0.997
  down to 0.603 to 0.759. Over 50 seeds the median corrected macro F1 is
  0.450 to 0.558, so seed 42 overstates the corrected performance.

## Primary analysis (`ml_models.ipynb`, from "Primary analysis: nested cross-validation")

Nested cross-validation: stratified 5-fold repeated 10 times outside,
5-fold tuning on log loss inside, five seeds (42 is primary), 95%
bootstrap intervals. The primary model is logistic regression with no
imbalance correction. All numbers below are seed 42.

| Outcome | Model | AUROC (95% CI) | Macro F1 (95% CI) | Notes |
|---|---|---|---|---|
| Healthy vs sick | Logistic regression, no correction | 0.976 (0.949-0.995) | 0.948 (0.915-0.976) | Calibration intercept 0.02, slope 1.10 |
| Healthy vs sick | AST/ALT ratio only | 0.809 | | Baseline |
| Four classes | Logistic regression, no correction | 0.949 (0.908-0.978) | 0.647 (0.550-0.741) | Fibrosis recall 0.20 |
| Three stages | Logistic regression, no correction | 0.843 (0.758-0.916) | 0.681 (0.556-0.793) | Fibrosis recall 0.50 |

- Class weights and SMOTE-NC do not meaningfully improve discrimination
  and push the predicted risks too high (intercepts -1.65 and -1.41).
- At a cut-off of 0.1, logistic regression finds 94% of sick patients and
  flags 18 of every 100 patients. Class weights flag 32 per 100 for the
  same sensitivity, and have lower net benefit at every cut-off.
- At a realistic 2% prevalence, net benefit stays positive for the
  uncorrected model and turns negative for class weights.
- Results are stable over five seeds (AUROC within 0.02), similar in women
  and men, and slightly lower in patients under 50 (AUROC 0.955 against
  0.996).
- About 1 in 10 patients switches side of the 0.1 cut-off between
  bootstrap refits of the final model.
- Full details, with code locations, are in `RESULTS.md`.

## Exploratory single-split models (`ml_models.ipynb`, first part)

These come from the original COM725 approach and are superseded by the
primary analysis above. Three models (logistic regression, random forest and decision tree) were
trained in three ways, with the same settings and evaluation as the COM725
study: an 80/20 stratified split and a median imputer fitted inside each
pipeline, so the training rows alone set the median.

**Healthy or sick** (class weights, no SMOTE). The test set has 107 healthy and 15 sick
patients.

| Model | Accuracy | Macro F1 | Sick patients found |
|---|---|---|---|
| Logistic regression | 0.98 | 0.94 | 13 of 15 |
| Random forest | 0.98 | 0.94 | 13 of 15 |
| Decision tree | 0.98 | 0.96 | 15 of 15 |

**Four classes** (SMOTE inside the pipeline, plus class weights, so the imbalance is corrected twice). The test set has 107 healthy,
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
- **COM725 timing problem:** the COM725 models also used RNA and ALT
  values measured weeks after the baseline biopsy (RNA 4, RNA 12, RNA EF,
  ALT at weeks 4 to 24) to predict the baseline stage. This is a timing
  leak. It made no difference there only because the data carry no
  signal (flat, uniform distributions and stages of almost equal size),
  so the Egyptian data are used only as a cautionary note.

## Limits

- The test set in `ml_models.ipynb` has only 15 sick patients and comes
  from a single split. One extra mistake changes sick recall by about 7
  points.
- The disease-class test sets are small: 4 to 6 patients per class in
  `ml_models.ipynb` and 1 to 5 in `data_leakage.ipynb`, so one patient
  changes a recall by about 17 points or more.
- There are only 21 to 30 patients in each disease class and no external
  dataset to validate on.
- The healthy donors and the patients were collected separately, so the
  12% prevalence and the high AUROC are a best case (spectrum bias).
- Lab units are not given in the UCI file; the SI units in Table 1 are
  inferred from the value ranges.
- Model settings are tuned only in the primary analysis; the single-split
  models above use fixed settings.

## References

- Chicco, D. and Jurman, G. (2021) 'An ensemble learning approach for
  enhanced classification of patients with hepatitis and cirrhosis',
  *IEEE Access*, 9, pp. 24485-24498.
- Pate, A., Riley, R.D., Collins, G.S. et al. (2023) 'Minimum sample size
  for developing a multivariable prediction model using multinomial
  logistic regression', *Statistical Methods in Medical Research*,
  32(3), pp. 555-571.
- Riley, R.D. et al. (2020) 'Calculating the sample size required for
  developing a clinical prediction model', *BMJ*, 368, m441.
- Riley, R.D., Van Calster, B. and Collins, G.S. (2021) 'A note on
  estimating the Cox-Snell R2 from a reported C statistic (AUROC) to
  inform sample size calculations for developing a prediction model with
  a binary outcome', *Statistics in Medicine*, 40(4), pp. 859-864.
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
