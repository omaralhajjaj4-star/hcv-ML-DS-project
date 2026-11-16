# Published approaches to the UCI HCV dataset (hcvdat0.csv): classification design and class imbalance handling

Scope note: about 13 tool calls were used. Several full texts were blocked (Nature 303 redirect, PMC reCAPTCHA, medRxiv/preprints.org 403, EKB reset, Uhasselt 503). Claims below come only from text actually read (UCI page, PLoS ONE full PDF, Bulucu et al. PDF, Zhen PDF) or from search-result snippets; snippet-only claims are marked [SNIPPET].

## Which papers benchmark classifiers on this dataset, and what do they report?

### Takeaway
Nearly every 2022-2025 paper found balances the data with SMOTE and reports 94-99% accuracy. None of the ones I could read shows evidence of keeping SMOTE inside the cross-validation folds, and the one I read in full (Das et al., 2025) balances before splitting. Reported numbers are therefore very likely inflated and not comparable across papers. I found no paper that reports per-class recall for Hepatitis/Fibrosis/Cirrhosis under leakage-free CV.

### Cited Findings
- UCI page: 615 instances, target `Category` = 0 Blood Donor, 0s suspect Blood Donor, 1 Hepatitis, 2 Fibrosis, 3 Cirrhosis; features Age, Sex, ALB, ALP, ALT, AST, BIL, CHE, CHOL, CREA, GGT, PROT; missing values present (page marks ALB, ALP, AST, CHOL; the rest of the table was not fully shown); CC BY 4.0; donated 2020; the UCI page does not give per-class counts. — [UCI HCV data](https://archive.ics.uci.edu/dataset/571/hcv+data) (2020)
- UCI-cited introductory paper: Hoffmann, Bietenbeck, Lichtinghagen, Klawonn (2018), "Using machine learning techniques to generate laboratory diagnostic pathways - a case study", J Lab Precis Med. UCI dataset citation: Lichtinghagen, Klawonn, Hoffmann (2020), DOI 10.24432/C5D612. — [UCI HCV data](https://archive.ics.uci.edu/dataset/571/hcv+data). I did not retrieve the 2018 paper's text or numbers.
- Per-class counts (75 patients = 24 Hepatitis, 21 Fibrosis, 30 Cirrhosis, plus 540 donor-type records incl. suspect) are repeated in secondary sources, not on UCI — [Bulucu et al. 2022](https://dergipark.org.tr/tr/download/article-file/2050402) (75 liver patients vs 540 donors); [SNIPPET, EKB paper](https://iiis.journals.ekb.eg/article_342004_f12d778c16fca4fb807697767a695f3a.pdf) (24/21/30).
- Foundational 2013 Lichtinghagen source: a search returned a 2013 Hannover ELF-score paper (400 healthy controls, 79 chronic HCV patients; cutoffs 7.7/9.8/11.3), PubMed 23523583. This is probably the J Hepatol ELF paper and is NOT clearly the origin of the 615-row table (that table is attributed to Hoffmann 2018 / Lichtinghagen 2020 on UCI). Treat the "2013 source" attribution as unverified. — [PubMed 23523583](https://www.pubmed.gov/23523583/) [SNIPPET]
- Das et al., PLoS ONE 20(3):e0319078, published 2025-04-02. Five classes (0 donor, 1 suspect, 2 hepatitis, 3 fibrosis, 4 cirrhosis). Pipeline: winsorization of outliers, SMOTE to equalize all classes (Fig 4), then 70/15/15 train/validation/test split with `train_test_split`; the SMOTE step is described in preprocessing (Sect 2.2.3) before the split, i.e. leakage-prone: synthetic neighbours of a test sample can sit in training. Also 5-fold CV and grid search used for tuning. Metrics: accuracy, macro precision/recall/F1, AUC, log-loss, MCC. Default models: RF 98.25%, SVM 98.50%, MLP 98.75%, XGBoost 97.25%, LR 92.50%, AdaBoost 27.25% (macro F1 19.77). Tuned: SVM 99.25%, RF 99.00%, MLP 99.00%, KNN 98.75%. Table accuracies are multiples of 0.25, implying about 400 test rows, i.e. a SMOTE-expanded dataset (consistent with about 2,665 rows balanced to 533 per class). No per-class recall reported in the text I read. — [Das et al. 2025](https://doi.org/10.1371/journal.pone.0319078) (PDF read in full)
- Das et al. survey of earlier work (secondhand): Lilhore et al. 2023 96.82% accuracy; Safdari et al. 2022 RF 97.29%, sensitivity 90.99%; Singh et al. 2022 LR 94.3% on a "two-part dataset approach"; Hira et al. SVM 99.1%; Syafa'a 2021 NN 95.12%; Li et al. 2022 "Cascade RF-LR (with SMOTE) using ABC algorithm" multiclass accuracy 94.5%. — [Das et al. 2025](https://doi.org/10.1371/journal.pone.0319078). I did not verify the original papers.
- Lilhore et al., Sci Rep 13 (Aug 2023), hybrid improved RF + SVM with Ranker feature selection + SMOTE: reported 96.82% accuracy (precision 98.93%) with SMOTE + feature selection versus 41.541% without SMOTE, so the gap conflates balancing and feature selection. Full text not retrievable (PMC/Nature blocked); the SMOTE timing, class set and CV scheme are UNVERIFIED. — [Lilhore et al. 2023](https://www.nature.com/articles/s41598-023-36605-3.pdf), [PMC10394001](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10394001/) [SNIPPET]
- Bulucu, Acer, Latifoglu, Icer (2022, Turkish journal, accepted 2022-05-31): this is a BINARY task, 75 liver patients (HCV, fibrosis, cirrhosis merged) vs 540 healthy donors. SMOTE applied "before the classification stage" in WEKA as preprocessing, then 5- and 10-fold CV on the balanced data (leakage-prone). Best LightGBM 98.8% accuracy, 98.1% precision, 99.4% recall, kappa 0.98 (10-fold); RF, J48, AdaBoost, GBC also compared. — [Bulucu et al. 2022](https://dergipark.org.tr/tr/download/article-file/2050402) (PDF read, abstract and methods)
- Zhen (conference paper, around 2022 per a SCITEPRESS file dated Aug 2022; year/venue not confirmed): multinomial logistic regression with LASSO/ridge/stepwise on the HCV data; reports average accuracy and sensitivity for cirrhosis 99% and specificity for fibrosis 95%. The abstract states one-vs-rest style per-class metrics; no SMOTE mention found in the text I read. Per-class "accuracy 99%" on a rare class is trivially high due to true negatives. — [Zhen](https://www.scitepress.org/PublishedPapers/2022/112313/112313.pdf)
- A 2025 comparison of Naive Bayes vs RF on the 615 records with SMOTE exists [SNIPPET only]; an International Integrated Intelligent Systems (EKB) paper reports RF 83.70% accuracy as best model [SNIPPET only, probably without heavy balancing] — [EKB](https://iiis.journals.ekb.eg/article_342004_f12d778c16fca4fb807697767a695f3a.pdf)
- A practitioner walkthrough on a single train/test split reports RF recall of about 0.25 for Hepatitis and 0.20 for Fibrosis, i.e. what honest per-class numbers look like without resampling; secondary evidence only. — [danokeefe.github.io/HCV](https://danokeefe.github.io/HCV.html) [SNIPPET]
- Other leads found but NOT read: medRxiv 2024 (SMOTE + evolutionary-optimized neural network) [medRxiv 2024.03.05.24303786](https://medrxiv.org/content/10.1101/2024.03.05.24303786v1.full.pdf); preprints.org 2023 [202310.0952](https://preprints.org/manuscript/202310.0952/v1/download); preprints.org 2026 [202602.0659](https://www.preprints.org/manuscript/202602.0659/v1); Uhasselt PDF [documentserver.uhasselt.be](https://documentserver.uhasselt.be/bitstream/1942/47208/1/1b957323-8d98-482a-8c51-79176ed43756.pdf); Carnazzo et al., Clin Biochem 2025 [publicatt.unicatt.it](https://publicatt.unicatt.it/bitstream/10807/301216/1/Carnazzo%20V%2c%20Clinical%20Biochemistry_2025.pdf). Relevance not established.

### Inferences
- Suspect donors: Das et al. keep them as a separate class (5 classes); other papers appear to merge or drop them (one snippet reports 7 suspect cases removed). Class sets differ, so accuracies are not comparable.
- AdaBoost at 27% accuracy next to 98% for others in the same pipeline (Das) suggests the SMOTE-balanced data is easy for tree/kernel models, consistent with near-duplicate synthetic points shared across splits.
- A SMOTE-inflated number such as 98-99% for 5 classes is hard to reconcile with the 2018 original authors' difficulty separating Hepatitis from donors on lab values (the practitioner recall of 0.2-0.25 is the realistic baseline); this is my inference, not a sourced claim.

### Gaps
- Could not read Lilhore 2023, the medRxiv 2024 preprint, or the 2026 preprint, so their SMOTE timing and metrics are unknown.
- No paper found reporting macro F1 / per-class recall for Hepatitis, Fibrosis, Cirrhosis under SMOTE-inside-CV or class-weighted CV.
- Original Hoffmann 2018 paper numbers not retrieved.

## Do any papers use two-stage / hierarchical / cascade designs, and does it beat single-stage?

### Takeaway
Direct precedent on the HCV UCI data is thin: one cascade (Li et al. 2022, "Cascade RF-LR") is mentioned secondhand. On other HCV liver-fibrosis lab data, multistage stepwise models are reported to beat single-stage ones, but on different datasets and endpoints.

### Cited Findings
- Hashem, Rasmy, Wahba, Shaker (2012), Comput Methods Programs Biomed 105(3):194-209: single-stage vs multistage stepwise models (NN, decision tree, LR, nearest neighbour) for fibrosis degree in chronic HCV from routine lab tests; multistage AUROC 0.874-0.974, described as higher than earlier reports. Dataset size not visible in the abstract page; not the UCI data. — [Maastricht CRIS record](https://cris.maastrichtuniversity.nl/en/publications/single-stage-and-multistage-classification-models-for-the-predict/) (2012). The "multistage beats single-stage" claim is the authors' own, not independently checked.
- FibroSteps (ElKamary et al., 2013): four-step scheme on 355 Egyptian HCV patients with METAVIR staging; first step F0/F1 vs F2-F4; final accuracy 94.9% training / 89.8% validation. — [SNIPPET via search, no direct page fetched](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC522750/) (the link given in the search result is a different FibroTest overview, so the FibroSteps citation is UNVERIFIED).
- Li et al. 2022 "Cascade RF-LR with SMOTE using ABC algorithm": multiclass accuracy 94.5% on HCV data — secondhand via [Das et al. 2025](https://doi.org/10.1371/journal.pone.0319078); original not read.
- Singh et al. 2022 used a "two-part dataset approach" (secondhand, same Das source); details unknown.

### Inferences
- Healthy-vs-disease then stage is a natural fit because the three disease classes (n=75) are separable from donors on AST/GGT/ALT/CHE fairly easily but are mutually similar; a hierarchical design would train the stage model only on 75 patients, making variance and CV design the main risk.
- No source shows a hierarchical design beating single-stage on UCI HCV under leakage-free evaluation; this appears to be an open gap.

### Gaps
- Could not find a paper that explicitly benchmarks single-stage vs two-stage on hcvdat0; Li 2022 and Singh 2022 originals not retrieved.

## Which imbalance techniques are used and which work best?

### Takeaway
SMOTE (plain, applied up front to the whole dataset) dominates. I found no evidence in readable sources of class weights, balanced RF, EasyEnsemble/RUSBoost, cost-sensitive learning or GAN synthesis applied to this dataset, so no basis to say which works best.

### Cited Findings
- Plain SMOTE to equal class sizes: [Das et al. 2025](https://doi.org/10.1371/journal.pone.0319078); binary SMOTE in WEKA: [Bulucu et al. 2022](https://dergipark.org.tr/tr/download/article-file/2050402); Ranker + SMOTE: [Lilhore et al. 2023](https://www.nature.com/articles/s41598-023-36605-3.pdf) [SNIPPET]; SMOTE + evolutionary NN: [medRxiv 2024](https://medrxiv.org/content/10.1101/2024.03.05.24303786v1.full.pdf) [SNIPPET].
- Lilhore: 41.541% without SMOTE vs 96.82% with SMOTE + feature selection (conflated). — [PMC10394001](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10394001/) [SNIPPET]
- Outlier winsorization before SMOTE (Das). — [Das et al. 2025](https://doi.org/10.1371/journal.pone.0319078)

### Inferences
- With n=21 Fibrosis, SMOTE's default k=5 neighbours interpolate among very few real points; synthetic points carry little new information, so reported gains likely reflect leakage rather than learning.
- The project can differentiate itself by comparing class weights, balanced RF, RUSBoost/EasyEnsemble and SMOTE strictly inside stratified repeated CV; no published comparison found.

### Gaps
- No readable source evaluates cost-sensitive, ensemble-undersampling or GAN methods on this dataset; I did not search GAN/CTGAN specifically.

## Are the reported numbers comparable, or are there red flags?

### Takeaway
No. Classes differ (binary vs 4 vs 5 class; suspect donors kept, merged or dropped), splits differ (single split, 5/10-fold), and the two papers I could read in detail apply SMOTE before splitting/CV. Majority class (donors, about 86-88%) alone gives about 87% accuracy, so any accuracy below about 90% is near the trivial baseline and accuracies of 98-99% after balancing are suspect.

### Cited Findings
- Das 2025: SMOTE before train/val/test split, headline 98-99%. — [Das et al. 2025](https://doi.org/10.1371/journal.pone.0319078)
- Bulucu 2022: SMOTE as preprocessing, then CV on balanced data, 98.8% accuracy. — [Bulucu et al. 2022](https://dergipark.org.tr/tr/download/article-file/2050402)
- Reported accuracies span 83.7% (EKB RF) to 99.25% (Das tuned SVM) depending on pipeline. — see links above
- Metric mix: accuracy dominates; Das gives macro P/R/F1, AUC, MCC but not per-class recall; Bulucu gives accuracy, precision, recall, kappa for the binary task; Zhen gives per-class sensitivity/specificity (rare-class "accuracy" inflated by true negatives).
- Majority fraction: 533 donors of 615 is 86.7% (computed from the task's stated counts; UCI page gives no per-class counts).

### Inferences
- Treat all SMOTE-before-split numbers as upper bounds, and do not compare them to leakage-free results from this project.

### Gaps
- Could not confirm leakage status for Lilhore 2023, Safdari 2022, Hira, Li 2022, medRxiv 2024.
