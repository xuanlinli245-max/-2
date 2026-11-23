# Model Card — Heart Disease Risk Screening

## 1. Model Details

- **Model name:** `heart-disease-voting-ensemble`
- **Version:** `1.1.0`
- **Model type:** binary classification; soft-voting ensemble
  (Logistic Regression + Random Forest + HistGradientBoosting)
- **Developer:** Xuanlin Li, educational ML project
- **Year:** 2026
- **Framework:** scikit-learn
- **Artifact:** full `Pipeline` (`model.joblib`), including preprocessing and classifier
- **License:** MIT (code); dataset — see the source page

### Architecture

```
raw patient record
  -> schema/range validation
  -> ZeroToNaN (Cholesterol=0, RestingBP=0 -> missing)
  -> numeric: median imputation + StandardScaler
  -> categorical: most-frequent imputation + OneHotEncoder
  -> soft voting: LogisticRegression (0.3) + RandomForest (0.4) + HistGradientBoosting (0.3)
  -> P(HeartDisease=1)
  -> threshold 0.107 -> risk band -> doctor review
```

## 2. Intended Use

**Primary use.** Support screening: estimate the probability of a heart-disease diagnosis from
11 routine clinical features and help prioritise which patients should be seen earlier.

**Primary users.** Clinician or nurse during intake; ML engineer for monitoring; researcher for
retrospective analysis.

**Out of scope.**
- diagnosis or treatment decisions;
- emergency triage;
- patients under 18;
- populations not represented in the training data;
- direct patient-facing use without a clinician.

The model is a **second opinion**, not a replacement for a doctor.

## 3. Inputs and Outputs

### Input features (11)

| Feature | Type | Values / range |
|---|---|---|
| Age | numeric | 18–100 years |
| Sex | categorical | M / F |
| ChestPainType | categorical | ATA / NAP / ASY / TA |
| RestingBP | numeric | 50–250 mm Hg; 0 = missing |
| Cholesterol | numeric | 0–700 mg/dl; 0 = missing |
| FastingBS | binary | 0 / 1 |
| RestingECG | categorical | Normal / ST / LVH |
| MaxHR | numeric | 60–220 bpm |
| ExerciseAngina | categorical | N / Y |
| Oldpeak | numeric | −3…+6; negative values are valid |
| ST_Slope | categorical | Up / Flat / Down |

### Output

- `P(HeartDisease=1)` — probability in [0, 1];
- `prediction` — 0/1 after the selected threshold;
- `risk_band` — low / medium / high;
- `threshold` and `model_version` — for audit and reproducibility.

The probability is **not calibrated** and must not be presented to a patient as a clinical risk
percentage.

## 4. Metrics

### Cross-validation (5-fold, training split)

| Metric | Mean ± std |
|---|---:|
| ROC-AUC | 0.931 ± 0.025 |
| Recall | 0.879 ± 0.034 |
| F1 | 0.872 ± 0.020 |

### Held-out test set (184 patients)

| Metric | @ threshold 0.50 | @ screening threshold 0.107 |
|---|---:|---:|
| ROC-AUC | 0.930 | 0.930 |
| PR-AUC | 0.937 | 0.937 |
| Recall | 0.863 | **0.971** |
| Precision | 0.907 | 0.739 |
| F1 | 0.884 | 0.839 |
| Accuracy | 0.875 | 0.793 |

Confusion matrix @ 0.107: `TN=47, FP=35, FN=3, TP=99`.

95% bootstrap CI @ 0.107: ROC-AUC `[0.891, 0.963]`, recall `[0.933, 1.000]`,
precision `[0.659, 0.805]`, F1 `[0.786, 0.884]`.

### Threshold choice

The threshold is a **product decision**, not a property of the model. For screening, maximise recall
subject to precision ≥ 0.70: threshold 0.107 catches 99 of 102 positives at the cost of 35 false
positives. The balanced operating point 0.50 gives 14 false negatives and 9 false positives.

## 5. Training Data

- **Source:** public heart-disease dataset (918 records, 11 features).
- **Target:** `HeartDisease` (0/1).
- **Split:** 734 train / 184 test, stratified, `random_state=42`.
- **Preprocessing:** fitted only on the training split, inside the Pipeline.
- **Fingerprint:** stored in `models/model_metadata.json`.

## 6. Evaluation Data

- Held-out 20% test set, not used for fitting or threshold selection.
- Threshold selected on out-of-fold predictions from the training split.
- No external validation cohort; this is a limitation.

## 7. Ethical Considerations

- Not a medical device; no CE/FDA approval.
- Risk of automation bias: the clinician must remain the decision-maker.
- No fairness audit by sex/age yet; this is a blocker before a clinical pilot.
- No data on smoking, medication, family history or socioeconomic factors.
- Patient data must be minimised, encrypted and processed under a DPIA.
- A high-risk result must be framed as “needs review”, not as a diagnosis.

## 8. Limitations

- 918 records from a single retrospective source; no external or temporal validation.
- Probabilities are not calibrated.
- No subgroup performance guarantee.
- Low threshold 0.107 is sensitive to distribution shift.
- Retrospective ROC-AUC does not guarantee prospective clinical benefit.

## 9. Maintenance and Monitoring

- **Triggers:** quarterly schedule, PSI ≥ 0.2, recall < 0.90, new data source, quality incident.
- **Release gates:** CV ROC-AUC ≥ 0.90; test recall ≥ 0.90; precision ≥ 0.70;
  subgroup recall gap ≤ 0.05; schema and leakage tests pass.
- **Rollback:** keep the previous artifact and metadata; switch the model path and restart the API.
- **Logging:** model version, threshold, input hash, prediction and human override.

## 10. Citation and Contact

Educational project. For questions, open an issue in the project repository.

**Dataset citation:** see `data/README.md` and the original Kaggle source page.
