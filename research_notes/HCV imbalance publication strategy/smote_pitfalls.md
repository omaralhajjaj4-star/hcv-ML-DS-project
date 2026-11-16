# Methodological pitfalls of SMOTE and other resampling/synthetic-data methods in clinical and tabular prediction

Note on verification: most claims come from search-result summaries and abstract-level fetches, not full-text reads. Items marked (unverified) should be checked against the full paper before citing a number.

## 1. SMOTE vs class weights vs threshold tuning vs cost-sensitive learning (2020-2026, tabular/medical, small minority)

### Takeaway
Recent evidence does not show SMOTE beating simpler options. Threshold adjustment is the most consistently effective single fix, and strong learners (RF, GBM) generally do not need balancing. No study I found directly tests the fewer-than-30-minority-cases regime with all three methods.

### Cited Findings
- van den Goorbergh et al. (JAMIA 2022, logistic regression): random undersampling, random oversampling and SMOTE gave poorly calibrated models with strongly overestimated minority-class probability, did not raise AUC, and similar sensitivity/specificity trade-offs were obtained by shifting the probability threshold instead. — [PMC9382395](https://pmc.ncbi.nlm.nih.gov/articles/PMC9382395); preprint [arXiv 2202.09101](https://arxiv.org/abs/2202.09101)
- Abdelhamid and Desai (arXiv 2024): SMOTE, class-weight tuning and decision-threshold calibration vs baseline, 15 models x 30 datasets (9,000 experiments, F1 primary). All three generally beat baseline, decision threshold calibration was "the most consistently effective", and the best method varied by dataset. Datasets had 500-20,000 rows and 1-15% rare class (larger than typical clinical cohorts). SMOTE vs class-weights ranking not stated in the abstract. Preprint, not peer-reviewed as far as I saw. — [arXiv 2409.19751](https://arxiv.org/abs/2409.19751)
- Elor and Averbuch-Elor (2022, 73 tabular datasets): balancing helps weak classifiers but not strong state-of-the-art ones; plain random oversampling performs about as well as SMOTE-like methods; metric choice and hyperparameter selection strongly affect conclusions. Abstract mentions "several other scenarios" where balancing helps, but I did not retrieve the list. — [arXiv 2201.08528](https://arxiv.org/abs/2201.08528)
- Andersen et al. (arXiv, Feb 2026), ten clinical datasets, 605,842 patients: SMOTE, RUS and ROS at 1:1 gave ROC-AUC changes of -0.01 (SMOTE, p<0.05), -0.004 (RUS, n.s.), -0.002 (ROS, p<0.05); Brier scores were worse (0.029-0.080 higher, p<0.05); calibration intercept and slope deviated markedly. Conclusion: "no positive impact on predictive performance". Preprint. — [arXiv 2603.00208](https://arxiv.org/abs/2603.00208)
- Tarawneh et al. (IEEE Access 2022, "Stop Oversampling for Class Imbalance Learning: A Review"): more than 70 oversampling methods on nine real datasets; concluded oversampling is unreliable and should be avoided in real-world applications, especially medical. Authors are oversampling-skeptical and the evidence is nine datasets (details from search summary). — [arXiv 2202.03579](https://ar5iv.labs.arxiv.org/html/2202.03579)
- Blagus and Lusa (BMC Bioinformatics 2013, foundational): on high-dimensional data SMOTE does not reduce bias toward the majority class for most classifiers, is less effective than undersampling, reduces variance and induces correlation between samples. Only k-NN (Euclidean) clearly benefits, after variable selection. Relevant mainly if feature count is large relative to n. — [PMC3648438](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3648438/)
- Fernandez, Garcia, Herrera, Chawla (JAIR 2018, SMOTE 15-year review): summative overview of SMOTE, extensions and open challenges. I could not read its conclusions section; use only as a general citation. — [doi 10.1613/jair.1.11192](https://www.doi.org/10.1613/jair.1.11192)
- A 2026 Informatica paper on small imbalanced tabular healthcare data argues standardized evaluation is lacking, so large-benchmark results may not transfer (from search snippet only; not opened). — [Informatica](https://www.informatica.si/index.php/informatica/article/view/12581/6709)

### Inferences
- For an HCV-style dataset with a small minority, the defensible baseline is: no resampling or class weights, strong learner, tune decision threshold on out-of-fold predictions, report SMOTE only as a sensitivity analysis inside CV folds.
- Caveat for the writer: benchmarks that favour resampling mostly use F1/accuracy-style threshold metrics, where threshold shifting would do the same job; the harm shows up in calibration and proper scoring rules.

### Gaps
- No head-to-head study of SMOTE vs class weights vs threshold with fewer than 30 minority cases found. Cost-sensitive learning vs SMOTE specifically in medical data was not separately searched.

## 2. Leakage from resampling before the split: size of optimism and meta-research

### Takeaway
Oversampling before splitting/CV causes large, quantified optimism (AUC up to +0.34 in a 15-dataset radiomics study), and individual published medical papers have been documented as affected. I found no formal systematic review that counts how many published medical ML papers did this.

### Cited Findings
- Demircioglu (Scientific Reports, May 2024): 15 radiomic datasets, 5-fold CV x 30 repeats. Oversampling applied to all data before CV produced bias up to +0.34 AUC, +0.33 sensitivity, +0.31 specificity, +0.37 balanced accuracy; about +0.10 AUC per increase in imbalance level; Brier score differed by up to -0.18; undersampling bias at most about 0.04 AUC. Worked example: correct AUC 0.71 vs leaked 0.87 (second figure from search summary). Mechanism: synthetic points land next to test samples. — [PMC11109211](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11109211/)
- Vandewiele et al. (Artificial Intelligence in Medicine 2021, vol. 111, 101987): re-examined near-perfect published results on the Term/Preterm EHG database; showed over-sampling before partitioning reproduces those results and that doing it after partitioning gives substantially lower AUCs. Exact numbers not retrieved (abstract only). — [arXiv 2001.06296](https://arxiv.org/abs/2001.06296)
- Blagus and Lusa (BMC Bioinformatics 2015): risk of overestimated accuracy is greater with oversampling; many published studies applying SMOTE before CV likely overoptimistic (from search summary; article not opened). — [doi 10.1186/s12859-015-0784-9](https://www.doi.org/10.1186/s12859-015-0784-9)
- Eltawil et al. (J. Cardiovasc. Dev. Dis., comment, ~Jan 2026) on Iacobescu et al. 2024: near-99% results likely from SMOTE-ENN applied before the train/test split. Authors' reply accepted that their wording implied global resampling and said leakage-free results are moderate. The 0.99 AUC figure comes from a third-party aggregator (unverified). — [Comment](https://pmc.ncbi.nlm.nih.gov/articles/PMC12842384/); [Reply](https://pmc.ncbi.nlm.nih.gov/articles/PMC12841616/) (PMC page returned a verification screen on fetch; details from search snippets)
- A review noted by a search result reports more than 9 published stroke-prediction papers oversampled before splitting (secondary web source, not verified): [stroke ML write-up](https://samhp.rbind.io/project/stroke-machine-learning)
- A recent arXiv migraine paper says one prior study applied SMOTE globally before partitioning, inflating accuracy (snippet only): [arXiv 2605.23453](https://arxiv.org/pdf/2605.23453)

### Inferences
- Optimism scales with imbalance and with how aggressively minority points are interpolated; with ~20 real minority cases, each synthetic point sits very close to real ones, so leakage would be severe.
- The leakage is a pipeline/reporting problem; reviewers can check it by asking whether resampling sits inside the CV loop.

### Gaps
- No systematic review quantifying prevalence of this error across medical ML literature found. Vandewiele before/after numbers and Blagus and Lusa 2015 numbers not extracted.

## 3. If calibrated probabilities are needed

### Takeaway
Do not resample; if you do, expect overestimated risk that recalibration may not fully fix. Evaluate with proper scoring rules and calibration plots.

### Cited Findings
- Carriero, Luijken, de Hond, Moons, van Calster, van Smeden (arXiv April 2024; published Statistics in Medicine 2025, 44(3-4), e10320 per search summary): Monte Carlo simulations over sample size, predictors, event fraction plus MIMIC-III illustration; models without correction had equal or better calibration in every scenario; corrections caused risk overestimation; recalibration "was not always able" to fix it; correction may be harmful for reliable individual risk estimates. Algorithm list not retrieved. — [arXiv 2404.19494](https://arxiv.org/abs/2404.19494)
- van den Goorbergh et al. 2022 (above): overestimated minority probability after SMOTE/RUS/ROS. — [PMC9382395](https://pmc.ncbi.nlm.nih.gov/articles/PMC9382395)
- Andersen et al. 2026: worse Brier, intercept and slope after resampling. — [arXiv 2603.00208](https://arxiv.org/abs/2603.00208)
- Demircioglu 2024 also shows Brier distortion under leakage. — [PMC11109211](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11109211/)

### Inferences
- Recommended reporting: calibration intercept, slope, calibration curve, Brier score, log-loss alongside AUC; threshold chosen on the original-prevalence data (for example via decision curve or cost ratio).

### Gaps
- No explicit published recalibration-after-resampling procedure with guaranteed success; the literature says it is only partial. Calibration of RF/GBM without resampling in tiny samples was not covered.

## 4. Very small minority classes (about 20 patients): validity, privacy, overfitting

### Takeaway
SMOTE adds no new information; with about 20 real cases the synthetic points are convex combinations of those same patients, so they cannot add independent evidence, and privacy and validity concerns apply. No paper I found studies exactly the n<30 case for SMOTE.

### Cited Findings
- Ganev et al., "SMOTE and Mirrors" (ICLR 2026): distance-to-closest-record and naive distinguishing checks miss leakage, while membership inference works with high accuracy; ReconSMOTE can reconstruct real minority records; authors argue leakage is inherent to interpolation, and minority records are the most exposed. — [arXiv 2510.15083](https://arxiv.org/abs/2510.15083v2); [ICLR listing](https://mlanthology.org/iclr/2026/ganev2026iclr-smote/)
- Blagus and Lusa 2013: SMOTE preserves class mean, reduces variability and induces correlation between samples. — [PMC3648438](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3648438/)
- Events-per-variable is a weak guide to overfitting; performance depends on number of predictors, sample size and event fraction (sample-size literature, e.g. [arXiv 2602.23507](https://arxiv.org/pdf/2602.23507), snippet only). A systematic review found oncology ML studies a median of 302 events below the minimum required sample size (snippet; source not opened).
- A 2025 arXiv paper reports a tabular-health augmentation scheme beating resampling-only (AUC, p=0.016) in small data; this is a method proposal from the authors of the scheme: [arXiv 2501.18741](https://arxiv.org/pdf/2501.18741) (snippet only).

### Inferences
- Effective sample size remains about 20; confidence intervals (bootstrap, repeated CV) should reflect that and not the inflated synthetic n.

### Gaps
- No empirical paper specifically on SMOTE with <30 minority cases; no clinical-validity guideline on synthetic minority patients found.

## 5. Mixed categorical and numeric features (SMOTENC)

### Takeaway
SMOTENC works but its handling of categories is crude, and critiques come mostly from authors of competing methods.

### Cited Findings
- SMOTE-NC treats all category mismatches as equally distant, using the median standard deviation of continuous features as penalty, and is undefined if there are no continuous features (Mukherjee and Khushi, SMOTE-ENC, 2021). — [arXiv 2103.07612](https://arxiv.org/abs/2103.07612v1); [MDPI](https://mdpi.com/2571-5577/4/1/18)
- A 2025 analysis (Sakho et al., per a secondary summary, unverified) reports generated categorical tuples are not always coherent and association with continuous features degrades as categorical dimension grows. — [emergentmind summary](https://www.emergentmind.com/topics/smote-nc)

### Inferences
- If the HCV dataset has few categorical columns (e.g. sex) and mostly continuous lab values, SMOTENC concerns are modest; interpolating continuous labs between patients can still produce physiologically implausible combinations (inference, not sourced).

### Gaps
- No independent benchmark of SMOTENC vs alternatives found.

## 6. Do strong classifiers need resampling?

### Takeaway
No for strong learners in most benchmarks (Elor and Averbuch-Elor), and resampling is more likely to help weak learners and threshold-based metrics.

### Cited Findings
- Elor and Averbuch-Elor 2022: balancing does not improve strong (state-of-the-art) classifiers; random oversampling is as good as SMOTE-like variants. — [arXiv 2201.08528](https://arxiv.org/abs/2201.08528)
- Andersen et al. 2026: no systematic discrimination gains across linear and several non-linear models. — [arXiv 2603.00208](https://arxiv.org/abs/2603.00208)
- Carriero et al. 2024/2025: miscalibration across multiple ML algorithms. — [arXiv 2404.19494](https://arxiv.org/abs/2404.19494)

### Gaps
- RF/GBM specific numbers not extracted; the "scenarios where balancing helps" list in Elor and Averbuch-Elor not retrieved.

## 7. Evaluating on a test set containing synthetic points; how to describe limitations

### Takeaway
Evaluating on synthetic or resampled test data is widely called invalid; test data should keep the natural prevalence and real patients only. I found no formal reporting standard for describing synthetic-data limitations, only practitioner and methods-paper guidance.

### Cited Findings
- Demircioglu 2024 shows the mechanism and size of inflation when synthetic points derived from all data reach the test fold. — [PMC11109211](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11109211/)
- Practitioner sources state to evaluate on original, imbalanced test data and never on a SMOTE dataset: [Don't Evaluate Your Model On a SMOTE Dataset (blog)](https://gmcirco.github.io/blog/posts/tiny-recid/recid.html); [KDnuggets](https://kdnuggets.com/why-most-people-misuse-smote-and-how-to-do-it-right). These are blogs, not peer-reviewed.
- A software-engineering paper argues that augmented variants of training samples in test sets inflate metrics: [arXiv 2502.01825](https://arxiv.org/pdf/2502.01825v1) (snippet only).
- Clinical prediction guide for clinicians warns data contamination can drastically inflate performance: [arXiv 2006.15069](https://arxiv.org/pdf/2006.15069).

### Inferences
- Likely reviewer challenges (inferred from the above): Was resampling inside each CV fold? Is the test set real-only at natural prevalence? Are calibration and Brier reported? Was a threshold-tuning/class-weight baseline compared? Does the effective sample size (about 20 events) support the conclusions, and do the confidence intervals reflect it? Are privacy implications of releasing synthetic minority records addressed (Ganev et al.)? A suitable limitations paragraph would state: synthetic points carry no new information, were used only in training, probabilities may be miscalibrated and were assessed on real data.
- Reporting frameworks such as TRIPOD+AI may be relevant but I did not verify what they say about resampling.

### Gaps
- No peer-reviewed paper dedicated to criticism of synthetic points inside test sets beyond leakage papers; no verified journal guidance wording on describing synthetic-data limitations.
