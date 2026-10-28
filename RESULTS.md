# Recorded results and reproducibility

This document separates historical notebook outputs from maintenance verification performed on 2026-10-28. Historical outputs and execution counts are preserved, including their original warnings and references. No recorded result was replaced by a new run. Display precision is retained below; extra decimals from new runs are not retroactively assigned to historical outputs.

## Configuration A: 30,000-row Excel experiment

All three notebooks use 23 predictors and the same stratified 80/20 outer split, seed 42 (24,000 training and 6,000 test rows). Default is the positive class. Metrics use threshold 0.50 except ROC-AUC, which summarizes ranking.

| Recorded model | Accuracy | ROC-AUC | Precision | Recall | F1 | Source |
| --- | --- | --- | --- | --- | --- | --- |
| Logistic regression | 0.809 | 0.710 | 0.692 | 0.244 | 0.361 | `notebooks/01_logistic_regression.ipynb`, cell 27, three-decimal display |
| Neural network | 0.817833 | 0.770647 | 0.660714 | 0.362472 | 0.468127 | `notebooks/02_neural_network.ipynb`, cell 18 |
| XGBoost | 0.817667 | 0.780463 | 0.660248 | 0.361718 | 0.467381 | `notebooks/03_xgboost.ipynb`, cell 19 |

Cell indices in this document are zero-based. XGBoost cell 24 contains a historical manual transcription of the Configuration A neural-network metrics; it is not a live comparison. The saved XGBoost output reports 284 best trees, validation log-loss 0.4291 and selected Excel index 24154 (ID 24155), probability approximately 0.700. Logistic regression explains test index 4387 and reconstructs probability 0.9927 algebraically; this is not the XGBoost applicant or a SHAP decomposition.

## Configuration B: 24,000-row PyCaret explainability experiment

This experiment excludes SEX and MARRIAGE, uses 21 predictors and a stratified 75/25 split, seed 42 (18,000 training and 6,000 test rows). **Its metrics must not be ranked against Configuration A as if they shared a test population and inputs.**

| Recorded model | Accuracy | ROC-AUC | Precision | Recall | F1 |
| --- | --- | --- | --- | --- | --- |
| Separate neural network | 0.819500 | 0.769196 | 0.668050 | 0.364253 | 0.471449 |

Source: `notebooks/04_neural_network_explainability.ipynb`, cell 18. All explanation methods below use this model and the same CSV applicant index 15110 (saved probability 0.685). This CSV index is not a common client identifier across files.

| Recorded explanation result | Value | Source cell | Scope |
| --- | --- | --- | --- |
| Kernel SHAP top-four overlap with first run | [4, 4, 4, 4, 4] | 33 | Five seeds, fixed background/model/applicant |
| Maximum SHAP attribution range across seeds | 0.012 | 33 | Rounded saved output, log-odds units |
| LIME mean pairwise top-four overlap | 2.61 / 4 | 39 | Twenty seeds, 5,000 samples each |
| SHAP/LIME top-four overlap | 2 / 4: LIMIT_BAL, PAY_0 | 42 | Absolute-attribution/weight ranks |
| Free DiCE probabilities | [0.449, 0.025, 0.300, 0.385, 0.032] | 45 | Five displayed counterfactuals, rounded |
| Constrained DiCE probabilities | [0.246, 0.320, 0.176, 0.212, 0.450] | 47 | Five displayed counterfactuals, rounded |

Global SHAP uses 200 sampled test cases, not the full test set. LIME mean weights exclude runs where a feature is absent from the top ten. Its stability summary averages all pairs, whereas SHAP compares with run zero. SHAP log-odds and LIME surrogate weights have different meanings; agreement is assessed by rank, not equal magnitude.

The final cell 49 retains manually supplied external-slide XGBoost probabilities 0.22 and 0.495. It has no recorded execution/output. The corresponding slides and model are absent, so these are unverified historical inputs, not reproduced cross-model results.

## Maintenance verification

Checks used the existing Windows environment, Python 3.14.7 and direct dependency versions in `requirements.txt`. The complete historical software/hardware environment is unknown. No new package installation or upstream dataset download was performed. `pip check` passed with no broken requirements; this is not proof that a fresh installation resolves identically.

Notebook code was evaluated sequentially in separate Python namespaces with the working directory set to `notebooks/`. IPython's `display` was supplied for notebook display calls. Figures used the noninteractive Agg backend; only directory creation and PNG saving were suppressed in memory. Modeling, preprocessing, splitting, hyperparameters and explanation computations were executed unchanged. Outputs were collected separately and were not saved into the notebooks. The optional `%pip` installation cell and the unverified final external-slide cell were skipped.

An initial plain-Python harness lacked notebook-provided `display` and stopped the logistic and XGBoost display cells after computing their metrics. Both notebooks were rerun with `IPython.display.display` supplied and completed successfully. This was a verification-harness issue, not a notebook computation change. GUI rendering and Jupyter restart/run-all were not independently tested.

| Check | Outcome |
| --- | --- |
| Included datasets | Read successfully; no missing values; CSV row membership in Excel verified after schema/type normalization |
| Dataset preservation | SHA-256 unchanged for both files |
| Recorded notebook output preservation | Every cell output and execution count unchanged, verified by digest |
| Executable source preservation | AST comparison against original notebooks: only the logistic data-loading cell changed; its path now resolves from `notebooks/`. Other changes are comments, markdown and kernel metadata |
| Notebook structure | All four notebooks passed nbformat validation during execution checks |
| Logistic regression | Completed; reproduced all recorded metrics at their displayed precision and local probability 0.9927 |
| Configuration A MLP | Completed; reproduced metrics to six decimals and 31 epochs |
| XGBoost | Completed, including wrapper check; historical training outcome not exactly reproduced; details below |
| Configuration B MLP | Completed; reproduced all metrics to six decimals, 42 epochs and applicant 15110 |
| Kernel SHAP and LIME | Completed local/global computations and repetitions; reproduced recorded top-four agreement summaries and SHAP maximum range at displayed precision |
| DiCE | Both searches completed with five threshold-crossing cases each; free probabilities reproduced to three decimals, constrained probabilities differ slightly (below) |
| Fresh environment installation | Not attempted |
| External-slide XGBoost experiment | Not reproducible from repository evidence; excluded |

### XGBoost discrepancy: retained historical evidence

The unchanged current code produced the following verification metrics, separately from the historical table above:

| Verification Accuracy | ROC-AUC | Precision | Recall | F1 |
| --- | --- | --- | --- | --- |
| 0.818167 | 0.780371 | 0.664804 | 0.358704 | 0.465981 |

Early stopping selected 328 trees, validation log-loss 0.428844, and Excel index 4366 (ID 4367), probability 0.7009725. The result was consistent in two local verification executions. The original outputs record 284 trees and a different applicant. The historical environment and full execution provenance are unavailable; the cause of the discrepancy is unresolved. No hyperparameters were changed to force agreement, and the new metrics were not substituted for the originals. Any future XGBoost explanation must identify which fitted run and applicant it explains.

### Explainability verification details

The Configuration B run reproduced SHAP top-four overlaps `[4, 4, 4, 4, 4]`, maximum range 0.012049 (0.012 at saved precision), LIME pairwise overlap 2.610526 (2.61 at saved precision), and SHAP/LIME overlap 2/4. This confirms the reported summaries, not bitwise identity of every attribution or plot. The model trained for 42 epochs and selected applicant 15110, as recorded.

Free DiCE verification probabilities were `[0.449, 0.025, 0.300, 0.385, 0.032]`, matching the saved three-decimal output. Constrained probabilities were `[0.246, 0.321, 0.176, 0.210, 0.450]`, versus recorded `[0.246, 0.320, 0.176, 0.212, 0.450]`. All five still cross below 0.50, but exact numerical reproduction is not established. This difference occurred with the existing fixed seed and unchanged code; its cause remains unresolved. The verification took approximately 165 seconds for notebook 04, including approximately 138 seconds for constrained search, and historical outputs were preserved.

### Notebook name mapping

| Original name | Current name |
| --- | --- |
| `Model_1_LogReg.ipynb` | `01_logistic_regression.ipynb` |
| `neural_network.ipynb` | `02_neural_network.ipynb` |
| `xgboost.ipynb` | `03_xgboost.ipynb` |
| `NN_explainability.ipynb` | `04_neural_network_explainability.ipynb` |

Numbering indicates a reading order, not a shared execution session. Kernel metadata now uses the standard `python3` kernel identifier; users must select their installed environment. No model parameters were changed.

### Dataset fingerprints

```text
credit.csv
7cb96a9bd541d5e574ba44a30dd93e8d3448557899d3751cf9f6035a166775c2

default of credit card clients.xls
d480be11fcbabf6c99247ebb7d12f2cad68ad08570ccdafcbfcb0fb17c360211
```

## Unresolved scientific questions

- The SHAP correlated-feature attribution experiment is proposed but absent. Repeating seeds with a fixed background tests estimation variation only.
- Explanation stability and agreement are evaluated for one selected true-positive applicant. This selection does not establish behavior on errors, other risk levels or the full population.
- Matching seeds, equal test sizes or matching SHAP units do not establish cross-configuration comparability. A shared experiment would require a separately approved scientific extension.
- Preprocessing is fitted on the full outer training partition before inner early-stopping validation is separated. Outer test data remain excluded, but inner validation influences preprocessing statistics. The MLP and XGBoost also stop on different criteria (accuracy versus log-loss).
- Logistic threshold advice derived from test-set exploration is not a validated operational policy. No calibration or explicit decision-cost analysis is established.
- Repayment codes -2 and 0 are ambiguous and treated numerically; rare categorical codes and correlated bill amounts complicate interpretation. Full one-hot category coefficients are not reference-group contrasts.
- There are 35 duplicate Excel rows excluding ID and 24 duplicate CSV rows. Their meaning and overlap across partitions remain unassessed; no rows were removed.
- Synthetic neighborhoods and independently altered counterfactual coordinates need not be realistic financial profiles. Historical repayment records are not directly actionable interventions. A finite random search failure does not establish impossibility.
- LIME local surrogate fidelity is not reported. Conditional feature-weight means and different SHAP/LIME stability denominators should be disclosed in any presentation.
- The applicant fallback in notebook 04 does not explicitly require probability >= 0.50, and its final slide-specific assertion can fail for a fallback applicant. It was not triggered in the checked run; selection logic is preserved.
- There is no temporal validation, causal identification, fairness evaluation or uncertainty analysis supporting production deployment or general superiority claims.
