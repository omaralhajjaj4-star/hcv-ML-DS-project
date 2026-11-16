# Reporting guidelines for ML prediction studies and two-stage vs single-stage classification (HCV imbalance publication strategy)

Verification note: most claims below come from search-result summaries and abstracts. Direct fetches of BMJ, PMC and medRxiv returned 403/429/reCAPTCHA, so the full TRIPOD+AI and PROBAST+AI item text was NOT read. Item numbers/wording marked "unverified" must be checked against the BMJ paper before citing.

Dataset context found: the repo's hcvdat0.csv matches the UCI "HCV data" (615 rows, 12 features, Category = Blood Donor / Suspect Blood Donor / Hepatitis / Fibrosis / Cirrhosis; CC BY 4.0; Lichtinghagen, Klawonn & Hoffmann 2020; intro paper Hoffmann et al. 2018) — [UCI](https://archive.ics.uci.edu/dataset/571/hcv+data). It is NOT the 1,385-row Egyptian biopsy dataset (UCI 503) — [UCI 503](https://archive.ics.uci.edu/dataset/503/hepatitis+c+virus+hcv+for+egyptian+patients). Distinction matters for baselines: the 615 set has no biopsy stage and the "Fibrosis/Cirrhosis" labels are clinical categories (verify label provenance in Hoffmann 2018).

## What must a TRIPOD+AI-compliant paper report about class imbalance, resampling, validation and performance uncertainty?

### Takeaway
TRIPOD+AI (BMJ 2024) is a 27-item checklist (plus 13-item abstract checklist) that supersedes TRIPOD 2015, and it has an explicit class-imbalance item (13) requiring authors to say why and how imbalance methods were used and any later recalibration. Exact wording of the validation/uncertainty items could not be verified.

### Cited Findings
- TRIPOD+AI: BMJ 2024;385:e078378, 27-item checklist plus expanded checklist and 13-item abstract checklist; replaces TRIPOD 2015 — [search summary of BMJ/EQUATOR/PMC](https://www.equator-network.org/?p=11341); full text [PMC11019967](https://pmc.ncbi.nlm.nih.gov/articles/PMC11019967/) (2024; not fetched).
- Item 13 (class imbalance) applies to development and evaluation; asks authors to explain why imbalance methods were used, how, and any later steps to recalibrate the model/predictions — [Research Square copy of checklist](https://assets-eu.researchsquare.com/files/rs-10145598/v1/58a8e20c02212cfa4dce640d.docx) (secondary copy; verify in BMJ Supp Table 2).
- Item 12e (performance measures: discrimination, calibration, clinical utility) and item 18 (open science: funding, COI, protocol, registration, data availability 18e, code availability 18f) — [journalmetrics blog summary, secondary](https://www.journalmetrics.org/blog/tripod-ai-clinical-prediction-model-reporting) and the [ResearchGate copy](https://www.researchgate.net/publication/379887244_TRIPODAI_statement_updated_guidance_for_reporting_clinical_prediction_models_that_use_regression_or_machine_learning_methods) (2024). Unverified numbering.
- Imbalance correction evidence that reviewers using TRIPOD+AI may cite: random under/oversampling and SMOTE gave poorly calibrated models (minority probability strongly overestimated), no AUC gain; threshold shifting gave similar sensitivity/specificity trade-off; "imbalance is not a problem in itself" — [van den Goorbergh et al., JAMIA 2022](https://pmc.ncbi.nlm.nih.gov/articles/PMC9382395). A 2026 arXiv follow-up on published clinical models reports similar (no generalizable discrimination gain, degraded calibration) — [arXiv 2603.00208](https://arxiv.org/pdf/2603.00208) (not read in full).
- Leakage: applying SMOTE (or SMOTE-ENN) before the train/test split inflates results; synthetic points near-duplicate test points; correct practice is split first, resample inside training folds only (imblearn Pipeline). Evidence is case studies/commentary, not a measured prevalence — [commentary on Iacobescu et al. 2024](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12842384/); [arXiv 2401.13796](https://arxiv.org/pdf/2401.13796); [Hayat & Magnier, Mathematics 2025 (via summary)](https://mdpi.com/2227-7390/13/16/2563).
- Sample size: Riley et al. BMJ 2020 criteria for binary/continuous/survival; multinomial extension (Pate, Riley, Collins et al., Stat Methods Med Res 2023) bases sample size on Cox-Snell R2 of one-vs-one sub-models; pmsampsize R package does not cover multinomial — [Utrecht record](https://researchinformation.umcutrecht.nl/en/publications/minimum-sample-size-for-developing-a-multivariable-prediction-mod-3/); [pmsampsize](https://cran.r-project.org/web/packages/pmsampsize/refman/pmsampsize.html).
- Adherence of ML prediction studies to TRIPOD is generally poor — [systematic review, medRxiv 2021](https://medrxiv.org/content/10.1101/2021.06.28.21259089v1.full.txt).

### Inferences
- A compliant paper on the 615-row set should state: prevalence per class, whether/why resampling was used, that it was inside CV folds, report calibration (curves, slope, intercept/ECE) both with and without resampling, and fixed-threshold vs tuned-threshold results. A "no resampling + class weights/threshold tuning" arm is the defensible comparator given van den Goorbergh.
- With 615 rows and roughly 75 patients in the three disease classes (see below), a pre-specified sample-size justification will likely show the data is below recommended size for many predictors; say so as a limitation rather than omit.
- Report CIs by bootstrap or repeated stratified CV (stated as inference; no source fetched for specific method).

### Gaps
- Verbatim TRIPOD+AI items on validation design, performance uncertainty (CIs), fairness, and sample size not verified (page fetch blocked).
- Class counts for UCI 615 are not on the UCI page; a secondary source says 540 blood donors vs 75 liver patients — [search hit summarising ijeecs/ekb papers](https://ijeecs.iaescore.com/index.php/IJEECS/article/download/37072/19342); compute from hcvdat0.csv instead.

## PROBAST+AI, DOME, CLAIM

### Takeaway
PROBAST+AI (BMJ 2025) is the risk-of-bias tool reviewers apply to your paper; DOME (Nature Methods 2021) is a biology-oriented Data/Optimization/Model/Evaluation checklist well suited to a public-notebook benchmark; CLAIM is imaging-specific and mostly not applicable.

### Cited Findings
- PROBAST+AI: Moons et al., BMJ 24 Mar 2025, 388:e082505; two parts (development, evaluation), four domains each (participants and data sources, predictors, outcome, analysis); 16 signalling questions for development, 18 for evaluation; applicability rated for the first three domains; intended to replace PROBAST for regression or AI models — [Oxford ORA record](https://ora.ox.ac.uk/objects/uuid:8b15c087-459b-43fa-ad03-1f66ed6ba370) and [Birmingham record](https://research.birmingham.ac.uk/en/publications/probastai-an-updated-quality-risk-of-bias-and-applicability-asses/) (2025; abstract-level only).
- DOME: Walsh et al., Nature Methods 2021;18:1122-1127, doi 10.1038/s41592-021-01205-4; ELIXIR ML Focus Group; Data, Optimization, Model, Evaluation; question-style checklist (Box 1) to put in Methods/supplement; supervised ML focus; reporting only, no new methods — [arXiv preprint 2006.16189](https://arxiv.org/abs/2006.16189?context=cs); [Unibo PDF](https://cris.unibo.it/retrieve/e1dcb338-310e-7715-e053-1705fe0a6cc9/s41592-021-01205-4.pdf).
- CLAIM 2024 update: Tejani, Klontzas, Kahn et al., Radiology: AI 2024;6(4), doi 10.1148/ryai.240300; Delphi-based; adds Not Applicable option; prefers "external testing" and "reference standard"; scope is medical imaging AI, not radiomics biomarkers — [EQUATOR](https://www.equator-network.org/reporting-guidelines/checklist-for-artificial-intelligence-in-medical-imaging-claim-a-guide-for-authors-and-reviewers/); [PMC11304031](https://pmc.ncbi.nlm.nih.gov/articles/PMC11304031).

### Inferences
- For tabular lab data, use TRIPOD+AI as primary, self-assess with PROBAST+AI, and attach a DOME table as supplement. CLAIM is not appropriate (no imaging).
- Typical PROBAST "analysis" domain flags: small events per candidate predictor, missing-data handling, resampling leakage, no calibration, no external validation — plausible given the dataset, but specific signalling-question wording not verified.

### Gaps
- PROBAST+AI signalling questions text not read; DOME item list not read (only structure).

## Two-stage / hierarchical vs single multiclass: evidence, criticisms, evaluation

### Takeaway
Evidence that two-stage designs improve minority recall exists only as scattered case studies; error propagation (stage-1 errors cannot be fixed) is the standard criticism and was found to make hierarchical worse than flat in one study. I found no controlled multi-dataset medical comparison and no liver-fibrosis cascade ML study.

### Cited Findings
- Hierarchical decomposition used to cope with imbalance: CKD staging split into (stage 3) vs (stages 4+5), then finer split, aimed at preserving minority accuracy — [arXiv 1712.01931](https://arxiv.org/pdf/1712.01931) (2017; older). Oncology text study: hierarchical ClinicalBERT beat flat on rare subtype, F1 0.02 -> 0.06 (tiny absolute gain) — reported in search summary, source not individually identified; low confidence.
- Counter-evidence: Arabic medical question classification shared task, hierarchical F1 0.332 worse than flat due to error propagation; stage 1 ~70% accuracy so ~30% lost before stage 2; some minority classes never predicted — [ACL Anthology 2026.abjadnlp-1.20](https://aclanthology.org/2026.abjadnlp-1.20.pdf) (2026; single workshop study, non-liver).
- Regrouping (merging majority classes to balance multiclass) as alternative, with warning that accuracy misleads and precision/recall/F1 should be reported — [arXiv 2210.12234](https://arxiv.org/pdf/2210.12234) (2022).
- Evaluation: Kosmopoulos et al., Data Mining and Knowledge Discovery 2015;29:820-865 unify hierarchical measures into pair-based and set-based families; criticise set-based hierarchical P/R/F for over-penalising by counting too many ancestors; propose LCA-based F-measure (FLCA) and MGIA — [arXiv 1306.6802](https://arxiv.org/pdf/1306.6802). Designed for large text hierarchies, not 4-5 class lab data.
- Silla & Freitas hierarchical-classification survey: not found/verified.

### Inferences
- Two-stage helps minority recall mainly when stage 1 has very high sensitivity for "any disease" (so few disease cases are lost) and the stage-2 classifier faces a more balanced problem; with ~75 disease cases, stage 2 trains on very few samples, so variance is likely to dominate. This is reasoning from the cited cases, not a documented finding.
- Error-propagation arithmetic: end-to-end recall for class k = P(stage1 passes k) x P(stage2 correct | passed). Report this product, not stage-wise metrics.
- End-to-end evaluation recommendation (inference): evaluate the full cascade on the same outer folds as the flat model, with resampling/thresholds fitted inside training folds for both stages; report per-class recall, macro-F1, balanced accuracy, MCC, confusion matrix, stage-1 miss counts per class, paired bootstrap or repeated-CV CIs for the cascade-vs-flat difference; hierarchical-F is optional given a one-level hierarchy.

### Gaps
- No controlled medical study isolating two-stage vs flat across datasets was found.
- No two-stage ML paper for fibrosis staging from labs found; search returned single-stage ML vs APRI/FIB-4 only.
- Whether the Hoffmann 2018 / Egyptian-dataset papers use cascades: unverified.

## Are APRI and FIB-4 the expected baselines, and what do they achieve?

### Takeaway
Yes, in hepatology APRI and FIB-4 are the standard comparators for lab-based fibrosis/cirrhosis ML, typically AUROC ~0.7-0.85 vs biopsy; but they target biopsy-staged fibrosis in known HCV patients, and the 615-row set needs AST, platelets (absent) for APRI/FIB-4 — check feasibility.

### Cited Findings
- Cochrane review: FIB-4 and Forns, 62 studies (~100,000 participants) vs biopsy in hepatitis C; original cutoffs 1.45 (rule out >= F3) and 3.25 — [Cochrane CD011929](https://www.cochrane.org/CD011929/LIVER_how-accurate-are-fib-4-score-and-forns-index-non-invasive-tests-diagnosing-liver-fibrosis-scarring) (pooled numbers not retrieved).
- Significant fibrosis (>=F2) biopsy-based AUROC APRI 0.809 vs FIB-4 0.803 — [Concordance analysis, PMC7550034](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7550034/).
- Thai cohort: FIB-4 >=1.45 for cirrhosis sens 52.4%, spec 91.0%, AUROC 0.829; APRI AUROC 0.84 vs FIB-4 0.80 for significant fibrosis; APRI>=1 for cirrhosis sens 70.1%, spec 80.6%, AUROC 0.834; spec 96.3% at APRI>=2 — [PMC6023204](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6023204/) (stratified by AST; attribution of Thai cohort to that paper is from the search summary, verify).
- US cohort: FIB-4 1.45 sensitivity 82%, specificity 58% for F3-4 — same search results, [PMC7550034](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7550034/) (verify attribution).
- 69,106 Egyptian adults: FIB-4 AUC 0.68, APRI 0.58, low accuracy and high disagreement with biopsy; ML then proposed — [PMC8748759](https://pmc.ncbi.nlm.nih.gov/articles/PMC8748759) (identification from search summary).
- Pediatric Egyptian HCV (~166 children): RF AUCs 0.903/0.894/0.822 vs APRI/FIB-4 ~0.77-0.83 — [PMC6689505](https://pmc.ncbi.nlm.nih.gov/articles/PMC6689505) (verify).
- ML vs APRI/FIB-4 elsewhere: FibroBox (HBV) AUROC higher than TE, APRI, FIB-4 — [PMC7520974](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7520974/); FibroPredict sens 96.8%, NPV 90.9% vs FIB-4 — via [EMJ](https://www.emjreviews.com/hepatology/news/machine-learning-in-liver-fibrosis-diagnosis-and-progression/).
- AST/necroinflammation can make APRI/FIB-4 overestimate stage — PMC6023204 above.
- Hoffmann 2018 (the dataset's introductory paper): partykit decision trees from lab data, leave-one-out validation, accuracy significantly above guessing; ML supports but doesn't replace experts — [JLPM article 4401](https://jlpm.amegroups.org/article/view/4401). Numeric results not retrieved.
- Egyptian UCI 1,385-row dataset: a 2020 medRxiv preprint applied nine ML algorithms on demographics and routine labs — [medRxiv 2020.11.02.20224840](https://medrxiv.org/content/10.1101/2020.11.02.20224840.full.pdf) (not read, 403).

### Inferences
- APRI = (AST/ULN)/platelets x100; FIB-4 = age x AST/(platelets x sqrt(ALT)). The UCI 615 columns (ALB, ALP, ALT, AST, BIL, CHE, CHOL, CREA, GGT, PROT, Age, Sex) contain no platelet count (from the UCI feature list in search summary; confirm in the csv). If so APRI/FIB-4 cannot be computed and this is itself a reportable limitation; a proxy comparator (e.g. AST/ALT ratio, De Ritis) could be used and labelled as such.
- The task (blood donor vs disease vs stage) differs from APRI/FIB-4's validated task (stage within HCV), so comparisons should be framed as context, not head-to-head.

### Gaps
- Pooled Cochrane sensitivity/specificity not retrieved. No source found reporting APRI/FIB-4 on the UCI 615 data. Published accuracies for ML on UCI 615 not collected.

## Venues and reviewer expectations for small public-dataset benchmarking papers

### Takeaway
I found no source on venue acceptance policies; only general evidence on what reviewers/critics demand (external validation, clinical utility, TRIPOD adherence).

### Cited Findings
- Only 6% (31) of reviewed ML studies did external validation — [PMC11515850](https://pmc.ncbi.nlm.nih.gov/articles/PMC11515850) (summary; context uncertain).
- COVID-19 ML review of 62 papers: none of potential clinical use due to methodological flaws and bias, public "Frankenstein" datasets criticised — [AuntMinnie](https://www.auntminnie.com/clinical-news/digital-x-ray/article/15628012/frankenstein-datasets-other-flaws-hamper-covid-19-ai-studies).
- Clinical utility assessment is neglected — [2026 editorial, IJMI (FBK repository)](https://cris.fbk.eu/retrieve/8d1cf728-462b-4837-93a0-369668e908ec/1-s2.0-S1386505626002789-main.pdf); external validation alone insufficient — [arXiv 2305.03219](https://web3.arxiv.org/pdf/2305.03219).
- Poor TRIPOD adherence in ML prediction papers — medRxiv 2021 above.

### Inferences
- A methods-focused framing ("effect of imbalance handling and cascade vs flat under leakage-free nested CV, with calibration") fits as a reproducibility/benchmark paper; claims of clinical utility should be avoided without external data. Candidate outlets (Scientific Reports, PLOS ONE, BMC Med Inform Decis Mak, JLPM, Diagnostics) are my suggestions only, not verified from sources.

### Gaps
- No evidence collected on specific journal policies, acceptance of small public-dataset papers, or typical reviewer comments for them. Not verified.
