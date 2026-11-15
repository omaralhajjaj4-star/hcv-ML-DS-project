# Class imbalance strategies: findings

## Question

The cleaned HCV data contain 608 patients: Healthy 533, Hepatitis 24,
Fibrosis 21 and Cirrhosis 30. Which strategy for handling the minority
classes gives the best balanced performance, and which should be carried
into the machine-learning stage? This is a strategy comparison using two
reference classifiers, not the final model study.

## Design

The analysis starts from the unimputed data (31 missing predictor values)
with the label cleaning of `cleaning.ipynb`. Predictors are the 11 numeric
variables plus Sex. Within each training fold, an `imblearn` pipeline
applies median imputation and scaling (numeric) and one-hot encoding (Sex),
then the sampler, then the classifier. Resampling and imputation therefore
never see test-fold patients. Evaluation uses 5-fold stratified
cross-validation repeated five times (25 test folds, seed 42), with
identical splits for every strategy. The smallest Fibrosis training count
in any fold was 16, enough for five-neighbour SMOTE. Classifiers were
logistic regression and a 300-tree random forest. Nine strategies were
compared: no handling (baseline), `class_weight="balanced"`, random
over-sampling, SMOTE, BorderlineSMOTE, ADASYN, SMOTETomek, SMOTEENN and
random under-sampling. No ADASYN fold failed (errors were set to raise).

## Key results (mean over 25 folds)

| Model | Strategy | Macro F1 | Balanced acc. | Healthy | Hepatitis | Fibrosis | Cirrhosis |
|---|---|---|---|---|---|---|---|
| LR | Baseline | 0.653 | 0.625 | 0.995 | 0.400 | 0.366 | 0.740 |
| LR | class_weight=balanced | 0.602 | 0.667 | 0.927 | 0.578 | 0.478 | 0.687 |
| LR | RandomUnderSampler | 0.608 | 0.701 | 0.894 | 0.604 | 0.606 | 0.700 |
| RF | Baseline | 0.590 | 0.566 | 0.996 | 0.258 | 0.258 | 0.753 |
| RF | SMOTE | 0.677 | 0.680 | 0.990 | 0.558 | 0.344 | 0.827 |
| RF | SMOTETomek | 0.679 | 0.683 | 0.990 | 0.566 | 0.344 | 0.833 |
| RF | SMOTEENN | 0.682 | 0.689 | 0.985 | 0.574 | 0.366 | 0.833 |

Columns Healthy to Cirrhosis are per-class recall. Standard deviations
across folds are about 0.05 to 0.10 for macro F1 and balanced accuracy.

## Ranking

- Logistic regression, macro F1: baseline 0.653, then BorderlineSMOTE
  0.617, SMOTE, SMOTETomek and RandomUnderSampler (each 0.608). Every
  strategy lowered macro F1.
- Logistic regression, balanced accuracy: RandomUnderSampler 0.701,
  class_weight 0.667, SMOTE 0.660; baseline last (0.625).
- Random forest, macro F1: SMOTEENN 0.682, SMOTETomek 0.679, SMOTE 0.677;
  baseline 0.590 (ninth), RandomUnderSampler 0.595 (eighth).
- Random forest, balanced accuracy: SMOTEENN 0.689, SMOTETomek 0.683,
  SMOTE 0.680; baseline last (0.566).

Paired against the baseline, the random forest gained 0.086 (SMOTE) to
0.091 (SMOTEENN) macro F1 with Bonferroni-adjusted p of 0.0003 or below.
For logistic regression, the macro-F1 loss was clearest for random
over-sampling (-0.065, adjusted p = 0.0057), while the balanced-accuracy
gain was clear only for under-sampling (+0.076, adjusted p = 0.0032). These
p-values are indicative only, because cross-validation folds share
training data and are not independent.

## Recommendation

For the random forest, use a SMOTE-family method. SMOTEENN, SMOTETomek and
SMOTE differ by at most 0.005 macro F1, far less than the fold SD, so plain
SMOTE is a sensible default, with SMOTEENN or SMOTETomek as secondary
options to be tuned in the ML stage. All three raised Hepatitis recall from
0.258 to 0.558-0.574 and Cirrhosis recall to 0.827-0.833 while keeping
Healthy recall at 0.985 or above. Random under-sampling and
class weights were clearly weaker for this model.

For logistic regression, `class_weight="balanced"` is the simplest
choice: it raises balanced accuracy to 0.667 and Hepatitis and Fibrosis
recall (0.578, 0.478) without creating synthetic patients. If sensitivity
to rare disease outweighs everything else, under-sampling gives the best
minority recall but discards most Healthy data.

## Trade-offs

Every strategy lowered Healthy recall for logistic regression (0.894 to
0.952 versus 0.995), meaning more healthy patients flagged for follow-up;
macro precision also fell (baseline 0.750, class_weight 0.584). Synthetic
samples are interpolations of at most 30 real patients and may not
represent real clinical variation. Instability is substantial: minority
recall SDs reach 0.14 to 0.29, so a single split could rank strategies
differently. SMOTETomek matched SMOTE exactly for logistic regression,
suggesting Tomek-link cleaning removed little.

## Limitations

Only two classifiers with fixed hyperparameters were used; the best
strategy may change with tuning. Fibrosis recall stayed below 0.61 for
every strategy, so the hardest class is not solved by resampling alone.
Metrics were averaged over folds rather than pooled, and no external
validation set exists. The confusion matrices in the notebook (first
repeat, one prediction per patient) are for visual inspection only.
