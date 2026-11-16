# Credible evaluation of clinical ML classification with tiny minority classes (about 20-30 per class, 75 positives, about 600 patients)

Verification note: most sources below were read only via search snippets or abstracts (full texts mostly not opened). Items marked [UNVERIFIED] come from background knowledge or were not confirmed in a fetched source.

## 1. What do 2020-2026 methodological papers recommend for model comparison and uncertainty when the minority class has fewer than 30 cases?

### Takeaway
No single paper targets "n<30 minority" directly (none found). The converging advice is: pre-specify the protocol, report pooled/aggregated estimates with bootstrap or nested-CV-based intervals (naive fold-SD intervals undercover), use variance-corrected or Bayesian correlated tests for model comparison, and describe model comparisons as indicative, not confirmatory.

### Cited Findings
- Standard CV confidence intervals can have coverage far below nominal because each point is used in both training and testing and fold accuracies are correlated, so the usual variance estimate is too small; a nested-CV scheme for variance estimation gives close to correct coverage in many examples. Bates, Hastie, Tibshirani, JASA 2023 (arXiv 2021, v4 2022) — [arXiv 2104.00673](https://arxiv.org/abs/2104.00673)
- Varoquaux (2018): neuroimaging-sized samples (about 100) give error bars around +/-10%, and the standard error across CV folds strongly underestimates them — [Varoquaux, via search summary of "Cross-validation failure"](https://arxiv.org/pdf/1706.07581v1.pdf) (only snippet seen)
- Varoquaux and Colliot (2023 book chapter, "Evaluating machine learning models and their diagnostic value") cover confidence intervals for performance metrics — [HAL record](https://hal-lirmm.ccsd.cnrs.fr/PRAIRIE-IA/hal-03682454v5) (specific small-sample passage not confirmed)
- A 2026 large-scale study of CI methods in medical imaging finds the aggregation strategy substantially affects CI reliability, and CI methods differ in reliability and precision — [arXiv 2601.17103](https://arxiv.org/pdf/2601.17103) (abstract-level only; imaging, not tabular)
- Nadeau-Bengio corrected resampled t-test: Dietterich (1998) showed the naive t-test on resampled results underestimates variance and inflates Type I error; Nadeau and Bengio (2003) correct it (rho=1/k for k-fold); Bouckaert and Frank (2004) give the repeated k-fold version. Python implementation: correctipy (two-model only); R: correctR — [correctipy docs](https://correctipy.readthedocs.io/en/latest/statistic_info.html), [correctR vignette](https://search.r-project.org/CRAN/packages/correctR/vignettes/correctR.html)
- Corrected variance is approx naive variance x (1/k + n_test/n_train) — [third-party calculator, low-quality source](https://metricgate.com/calculator/nadeau-bengio-corrected-resampled-t-test); equivalent to rho/(1-rho)=1/(k-1) for rho=1/k per correctR. Check the original paper before relying on the exact form.
- Benavoli et al., JMLR 2017: argues against null-hypothesis tests, proposes Bayesian correlated t-test for CV results (accounts for overlapping training sets) and a Bayesian signed-rank test replacing Wilcoxon for multi-dataset comparison — [search summary, evaluma docs](https://evaluma.readthedocs.io/en/latest/tutorials/frequentist_vs_bayesian.html)
- Demšar (2006): recommends correlated t-test for a single dataset, Wilcoxon signed-rank across datasets (secondary description only) — [see same evaluma page](https://evaluma.readthedocs.io/en/latest/tutorials/frequentist_vs_bayesian.html)
- Riley and Collins, Biometrical Journal 2023: bootstrap the whole model-building process to produce prediction/calibration instability plots and an instability index; instability in individual risks is often considerable and shows up as miscalibration in new data — [PMC10952221](https://pmc.ncbi.nlm.nih.gov/articles/PMC10952221/)
- TRIPOD+AI (BMJ 2024, 27-item checklist, supersedes TRIPOD 2015) is the reporting standard reviewers will check against — [search-result summary, BMJ 385:e078378](https://doi.org/10.1136/bmj-2023-078378) [DOI not fetched, UNVERIFIED]

### Inferences
- With about 20-30 minority cases, a fold-level test set contains only 2-6 minority patients (for 5-10 folds), so fold scores are extremely noisy; paired tests on those folds are underpowered and, uncorrected, anti-conservative.
- Using Wilcoxon across CV folds or repeats treats non-independent values as independent; the Nadeau-Bengio/Bayesian correlated t-test is the defensible variant, and even it is only approximate. Frame differences as exploratory and supply a paired bootstrap CI of the metric difference (my recommendation, not from a single source).
- Repeated CV reduces the variance due to the split, not the variance due to the finite sample; extra repeats do not shrink the CI of the true performance (consistent with Bates et al.).

### Gaps
- No paper found giving empirical coverage of bootstrap CIs specifically for n of about 20 minority cases. Not verified: Wilson/Clopper-Pearson intervals for per-class recall (standard binomial practice, [UNVERIFIED] with no source fetched; for 20 cases with 15 hits recall 0.75, the exact CI is roughly 0.51-0.91 from my calculation).
- Whether the sklearn docs endorse the corrected t-test: not confirmed in sources fetched (sklearn does have an example "Statistical comparison of models using grid search" that uses the corrected resampled t-test; [UNVERIFIED]).

## 2. Is nested CV needed when hyperparameters are tuned, and how large is the optimism if skipped?

### Takeaway
Yes if tuning or any data-dependent selection (features, resampling such as SMOTE, thresholds, model choice) uses the same data; optimism is often modest per single hyperparameter but grows with the number of configurations and shrinks with sample size. Reported magnitudes vary by setting.

### Cited Findings
- Cawley and Talbot, JMLR 2010: model-selection criterion variance causes over-fitting in selection; the resulting bias is often comparable to the performance differences between algorithms; applies to any selection process optimising a criterion over a finite sample — [JMLR](https://www.jmlr.org/papers/v11/cawley10a.html)
- Vabalas et al., PLOS ONE 2019: simulation shows plain K-fold CV gives strongly biased estimates in small samples (bias still evident at n=1000); nested CV and train/test split were unbiased regardless of n; feature selection on pooled data contributed more bias than parameter tuning — [PMC6837442](https://pmc.ncbi.nlm.nih.gov/articles/PMC6837442)
- A cited study reports AUC bias of 5-10% frequently for samples under 100 when tuning across many configurations (secondary, from search snippet; paper not identified clearly) — [arXiv 1708.07180 candidate, unconfirmed](https://arxiv.org/pdf/1708.07180)
- Vendor blog suggests optimism gaps of 0.02-0.07 and at least 3 inner folds — [MetricGate](https://metricgate.com/docs/nested-cross-validation-bias/) (low quality; treat as illustrative only)
- Bates et al. 2023 also uses nested CV, for variance estimation not just bias — [arXiv 2104.00673](https://arxiv.org/abs/2104.00673)
- Bootstrap optimism correction applied to the whole pipeline is the other accepted route, endorsed in the Riley and Collins stability framework (bootstrap repeats all model-building steps) — [PMC10952221](https://pmc.ncbi.nlm.nih.gov/articles/PMC10952221/)

### Inferences
- For a pipeline with SMOTE + RF tuning, put resampling inside each training fold (never before splitting) and use nested CV (e.g. 5x outer repeated, 3-inner) or report that the grid is tiny/pre-fixed. If tuning is skipped (defaults or a pre-specified fixed grid not chosen using test data), state this explicitly.
- Inner folds with about 15-20 minority cases split 3 ways leave about 5 per inner validation fold; tuning is itself noisy at this size, so prefer small grids.

### Gaps
- No source quantifying optimism for tuned random forests on about 600 rows with 75 positives. Leave-one-out and bootstrap .632+ as alternatives: not researched (LOO is high-variance and not stratification-friendly; [UNVERIFIED] general knowledge).

## 3. Which headline metric should be pre-specified for multiclass imbalance, and what are the weaknesses of macro F1 and balanced accuracy at small n?

### Takeaway
No consensus single winner. Pre-specify one threshold-based headline (macro-averaged recall/balanced accuracy or multiclass MCC) plus threshold-free and calibration measures; always report per-class recall/precision. STRATOS guidance says classification measures are improper at non-default thresholds, so they should not be the only evidence.

### Cited Findings
- Van Calster et al., Lancet Digital Health 2025 (arXiv 2412.10288, Dec 2024): 32 measures across five domains; recommended essentials are AUROC, a calibration plot, net benefit/decision curve and a plot of predicted-probability distributions per outcome; classification measures such as accuracy and F1 are improper for clinically relevant thresholds other than 0.5 or the prevalence — [arXiv 2412.10288](https://arxiv.org/abs/2412.10288v1) (via search summary)
- Opitz and Burst (arXiv 2019, rev 2021): two common macro-F1 formulas (mean of per-class F1 vs F1 of macro precision/recall) are usually not equivalent; can differ by up to 0.5 and change classifier rankings; one rewards skewed error distributions — [arXiv 1911.03347](https://arxiv.org/abs/1911.03347). Specify which formula (sklearn uses the mean of per-class F1).
- Chicco and Jurman, BMC Genomics 2020: MCC preferred over accuracy and F1 for binary imbalanced tasks; F1 changes under class swap, MCC does not — [PMC6941312](https://pmc.ncbi.nlm.nih.gov/articles/PMC6941312)
- Chicco, Totsch, Jurman 2021 (BioData Mining): high MCC implies high values on all confusion-matrix rates, which is not true of balanced accuracy, informedness, markedness or F1 — [PMC7863449](https://pmc.ncbi.nlm.nih.gov/articles/PMC7863449) (as summarised in search results)
- McDermott et al., NeurIPS 2024 (arXiv 2401.06091): AUPRC is not inherently superior to AUROC under imbalance and can favour improvements in subgroups with more frequent positives; AUROC weights all false positives equally, AUPRC weights them by inverse score likelihood — [arXiv 2401.06091](https://arxiv.org/html/2401.06091v4)
- Multiclass sample-size context: Pate et al., Stat Methods Med Res 2023, extends Riley criteria to multinomial models; sample size depends on events per category and grows with the number of categories — [arXiv 2207.12892](https://pdf.arxiv.org/pdf/2207.12892)

### Inferences
- Macro F1 and balanced accuracy weight each class equally, so a class with 20 cases contributes as much as one with 300, and its noisy recall sets the headline's variance; a swing of 2 patients in a 20-case class changes recall by 10 points. Both depend on the decision threshold/argmax, and neither assesses calibration.
- Practical pre-specification (my recommendation): headline = macro recall (balanced accuracy) or macro F1 with the formula stated, computed on pooled out-of-fold predictions; co-primary = per-class recall with exact CI; secondary = one-vs-rest AUROC (and AUPRC with its prevalence baseline shown), multiclass MCC, Brier/calibration. Do not report accuracy as a headline.
- AUPRC must be interpreted against prevalence per class (baseline = prevalence); avoid claiming AUPRC "fixes" imbalance (McDermott).

### Gaps
- No source found on small-n variance/bias of macro F1, balanced accuracy or multiclass MCC specifically. Cohen's kappa weaknesses (prevalence dependence): not researched, [UNVERIFIED].

## 4. How should per-class results and confidence intervals be reported?

### Takeaway
Report per-class counts, recall, precision, with intervals that reflect the tiny n; compute metrics on pooled out-of-fold predictions with bootstrap CIs; show the confusion matrix with raw counts; never report only a mean of fold scores with SD.

### Cited Findings
- Fold-SD intervals undercover; need variance-aware methods (nested CV or similar) — [arXiv 2104.00673](https://arxiv.org/abs/2104.00673)
- Standard error across folds strongly underestimates true error bars; typical error bars are about +/-10% at n of about 100 — [Varoquaux 2018](https://arxiv.org/pdf/1706.07581v1.pdf)
- Bootstrapping the entire modelling process quantifies instability of individual predictions and calibration — [Riley and Collins 2023](https://pmc.ncbi.nlm.nih.gov/articles/PMC10952221/)
- TRIPOD+AI expects full reporting of performance with uncertainty — [BMJ 2024](https://doi.org/10.1136/bmj-2023-078378) [unverified detail]

### Inferences
- Suggested table: class, n (cases), correct, recall with exact (Clopper-Pearson) or Wilson 95% CI, precision with CI, one-vs-rest AUROC with bootstrap CI (resampling patients, stratified by class, whole pipeline refit if feasible). Add paired-bootstrap CI for between-model differences.
- Pooled out-of-fold predictions let one compute metrics once with a single, larger denominator; per-fold averages of metrics are noisy and can be undefined (zero minority cases or zero predicted positives in a fold). Caveat: pooled predictions from different fold models are not from one model, and bootstrap on pooled predictions ignores training variance, so it is somewhat optimistic [UNVERIFIED reasoning, consistent with Bates et al.].
- Use repeated stratified CV (e.g. 5x10-fold) to average out split randomness, but do not treat repeats as independent samples.

### Gaps
- No peer-reviewed source fetched that prescribes a per-class reporting template; the template above is my synthesis.

## 5. Calibration and sample-size considerations (Riley, EPV, learning curves)

### Takeaway
Minimum sample size should follow the Riley criteria rather than EPV rules of thumb; with 75 positives, only a small number of predictor parameters is supportable, so multiclass ML with many features is likely below recommended sizes. Calibration should be reported with a plot, calibration slope/intercept and Brier score, acknowledging wide uncertainty.

### Cited Findings
- Riley et al., BMJ 2020 (368:m441): the simple 10 events-per-predictor-parameter rule is too simplistic; sample size should be tailored using criteria on shrinkage (overfitting), optimism in apparent fit, and precision of the overall risk (intercept); software: pmsampsize — [BMJ DOI](https://doi.org/10.1136/bmj.m441), [pmsampsize docs](https://rdrr.io/pkg/pmsampsize/man/pmsampsize.html) (full text not fetched; exact thresholds such as shrinkage of at least 0.9 are [UNVERIFIED] from memory)
- For binary outcomes the effective sample size is driven by the number of events (the smaller class) — [search summary of Riley et al.](https://keele-repository.worktribe.com/OutputFile/464599)
- Pate et al. 2023 for multinomial models: needs expected Cox-Snell R2 or c-statistic per sub-model, events per category, number of candidate predictor parameters; more categories need larger samples — [arXiv 2207.12892](https://pdf.arxiv.org/pdf/2207.12892); 2024 J Clin Epidemiol guide for multinomial models — [arXiv 2312.12008](https://arxiv.org/pdf/2312.12008)
- Van Calster et al. 2025: calibration plot (smoothed) is an essential reporting item; models can discriminate well but be miscalibrated — [arXiv 2412.10288](https://arxiv.org/abs/2412.10288v1)
- Calibration curves need sufficiently large samples (Van Calster-related paper, snippet only) — [Calibration: the Achilles heel](https://ouci.dntb.gov.ua/works/4Lopk167)
- Riley and Collins: instability plots demonstrate how unstable individual risks are in development samples — [PMC10952221](https://pmc.ncbi.nlm.nih.gov/articles/PMC10952221/)
- Learning curves: no source gathered. A 2026 paper on Brier score misconceptions was seen only as a reference title, not read.

### Inferences
- Rebalancing (SMOTE etc.) distorts predicted probabilities, so calibration should be checked on the original-prevalence out-of-fold predictions and recalibration considered (general knowledge; no source fetched).
- With classes of about 20-30 patients, calibration curves per class will be extremely noisy; report slope/intercept with CI and mark as exploratory.
- A learning curve (performance vs training fraction, with CIs) is a useful demonstration that the model has not plateaued, but I found no methodological reference to cite.

### Gaps
- Exact Riley thresholds and a worked pmsampsize calculation for this data were not obtained; recommend running pmsampsize (R) or the Pate multinomial tool with project-specific inputs.
- Cohen's kappa, leave-one-out and .632+ bootstrap evaluations, scikit-learn documentation pages, Raschka's model-evaluation series, and the 2026 Brier misconceptions paper were not retrieved.
