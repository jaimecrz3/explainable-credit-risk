# Explainable Credit Risk

MMF1922H · Explainable AI · University of Toronto

A default probability ranks risk without explaining a customer's score. Analysts also need to understand which inputs matter, whether the reasons are stable, and whether proposed changes are realistic. This four-person project studies those questions through logistic regression, XGBoost and a neural network, with predictive performance as context rather than the main objective.

## Research question

> As models get more opaque, 'trust me, it works' isn't good enough anymore — especially in finance. What tools (SHAP, LIME, counterfactuals) actually let us open the black box, and where do they still fall short?

We distinguish interpretation from post-hoc explanation, assess agreement and sampling stability, and examine whether changing a prediction implies a feasible intervention.

## Research design

Logistic regression provides a readable baseline equation. XGBoost captures nonlinear relationships through trees and lets us examine global importance. A neural network provides a separate setting for prediction-based SHAP, LIME and counterfactual experiments.

The data describe Taiwanese credit-card clients: six months of repayment status, bills and payments, plus credit limit and demographics. The target is next-month default. A probability of at least 0.50 defines a hypothetical high-risk/decline decision, not a deployed underwriting policy.

There are two experimental configurations:

| Setting | A: common modeling setup | B: network explanation study |
| --- | --- | --- |
| Included dataset | UCI Excel, 30,000 rows | PyCaret-source CSV, 24,000 rows |
| Original predictors | 23; exclude ID and target | 21; also exclude SEX and MARRIAGE |
| Stratified split | 80/20, seed 42 | 75/25, seed 42 |
| Training / test rows | 24,000 / 6,000 | 18,000 / 6,000 |
| Models | Logistic regression, XGBoost, original neural network | Separately fitted neural network |
| Completed analyses | Logistic interpretation; XGBoost global importance | Kernel SHAP, LIME, stability/agreement, DiCE |

Both setups standardize numeric inputs, one-hot encode categories and retain undocumented codes. PyCaret identifies the CSV source, not a training framework. The network notebooks remain separate because their experiments differ. Equal seeds and test sizes do not create identical observations; A/B metrics are not a controlled comparison. See [data provenance](data/README.md).

## What each explanation provides

| Approach | How it works and what it explains | Implemented for |
| --- | --- | --- |
| Intrinsic interpretation | Coefficients describe the equation; coefficient × transformed input gives a local term. Numeric effects use standard-deviation units; category effects require contrasts. | Logistic regression |
| Global feature importance | Gain measures average split improvement, weight split frequency, and cover split coverage. These summarize the ensemble, without signed effects or individual explanations. | XGBoost |
| Local and aggregated attribution | Kernel SHAP estimates contributions against a training-background expectation. Values are in log-odds here; aggregated magnitudes summarize a sample. | Configuration B network |
| Local surrogate explanation | LIME samples synthetic neighborhoods, weights them by proximity and fits a simpler model to the network's probabilities. Its coefficients describe that local approximation. | Configuration B network |
| Counterfactual search | DiCE searches for alternative input profiles that cross the classification boundary, first freely and then with selected variables and directions restricted. | Configuration B network |

XGBoost prepares a wrapper and a case but does not execute SHAP, LIME or DiCE. Its completed contribution is global importance. The logistic decomposition is not SHAP.

## Main empirical findings

| Evidence from saved outputs | Interpretation |
| --- | --- |
| Logistic PAY_0 coefficient 0.661; exp(coefficient) 1.937. Local probability reconstructed as 0.9927. | A one-standard-deviation increase multiplies fitted conditional odds by about 1.94, holding other inputs fixed. Transparency does not remove ambiguity from rare categories or correlated inputs. |
| XGBoost PAY_0 ranks 1/8/3 under gain/weight/cover; BILL_AMT1 ranks 10/1/11. | The answer to “which feature is important?” depends on the definition. Importance is neither an individual reason nor a causal effect. |
| Global SHAP: 200 sampled cases; largest mean absolute attribution is PAY_0, 0.601663. Locally it contributes +1.556 log-odds; default probability is 0.685. | Aggregated patterns and local contributions differ. A high-risk prediction also contains offsets. |
| Five SHAP runs: unchanged top four against run zero; maximum range 0.012. Twenty LIME runs: pairwise overlap 2.61/4. Between methods: 2/4, PAY_0 and LIMIT_BAL. | Agreement is partial. Different stability definitions, one model/background/applicant: these are not a universal explainer ranking. |
| All five free and five constrained DiCE examples cross below 0.50. Selected constrained probabilities include 0.246 after a large payment change and 0.212 after rewriting repayment status. | Successful model-input changes need not be affordable, financially consistent or actionable. Constraints expose assumptions but do not establish causal interventions. |

Local cases differ across model families; only the methods inside B explain the same network and applicant. LIME means omit absent features, and surrogate fidelity is unreported.

These tools provide complementary views of model behavior, not evidence of fairness or causality. The proposed SHAP correlated-feature stress test remains unimplemented.

## Results and reproducibility

[RESULTS.md](RESULTS.md) records predictive metrics, source cells, dataset fingerprints, previous execution checks and unresolved questions. The included notebooks preserve their saved outputs.

Previous checks reproduced logistic/network metrics at displayed precision and the main explanation summaries. XGBoost's historical fit and two constrained DiCE probabilities differed in verification; both versions remain documented. A fresh installation is unverified. The publication audit performed no new training or explanation runs.

Other limits include preprocessing fitted before inner early-stopping validation, ambiguous repayment codes, correlated histories, a selected local case and unassessed duplicate overlap across partitions. Operational threshold selection, calibration, temporal validation and fairness assessment would require further work.

## Repository and presentation

```text
README.md
RESULTS.md
requirements.txt
data/
    README.md
    default of credit card clients.xls
    credit.csv
notebooks/
    01_logistic_regression.ipynb
    02_neural_network.ipynb
    03_xgboost.ipynb
    04_neural_network_explainability.ipynb
presentation/
    final_group_presentation.pptx
```

Notebooks 01–03 cover A; notebook 04 contains B's complete explanation study. Run them independently. Figures and tables are embedded; XGBoost also generates an optional `figures/` directory.

The [final editable PowerPoint](presentation/final_group_presentation.pptx) presents the three model families in ten slides. Four consecutive speaker sections have 2:30 scheduled each. Embedded notes contain the spoken explanations and source details. The assignment's ten-minute maximum requires rehearsal; the schedule alone leaves no overrun buffer.

## Installation and execution

The dependency manifest pins direct packages from the checked Python 3.14.7 environment. It is neither a recovered historical environment nor a complete transitive lockfile. Use a separate environment and select its kernel explicitly.

From the repository root on Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m ipykernel install --sys-prefix --name explainable-credit-risk --display-name "Explainable Credit Risk"
.\.venv\Scripts\python.exe -m jupyterlab --notebook-dir=notebooks
```

On macOS or Linux:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m ipykernel install --sys-prefix --name explainable-credit-risk --display-name "Explainable Credit Risk"
.venv/bin/python -m jupyterlab --notebook-dir=notebooks
```

Select **Explainable Credit Risk** as the kernel, with `notebooks/` as its working directory for `../data/` paths. Restart between notebooks. Execute copies or avoid saving over recorded outputs; explanation searches can take several minutes.

The optional `%pip install -q xlrd` cell in notebook 01 is unnecessary after installing requirements. Notebook 04 retains a CSV download fallback, but the included file should be used for the documented experiment. Its final external-slide comparison cell is unexecuted and unverified; it is not part of the reported evidence.

## References

- Yeh, I. *Default of Credit Card Clients*, UCI Machine Learning Repository; provenance and attribution are in [data/README.md](data/README.md).
- Lundberg, S. M. and Lee, S. I. (2017). *A Unified Approach to Interpreting Model Predictions*.
- Ribeiro, M. T., Singh, S. and Guestrin, C. (2016). *“Why Should I Trust You?”: Explaining the Predictions of Any Classifier*.
- Wachter, S., Mittelstadt, B. and Russell, C. (2017 preprint). *Counterfactual Explanations without Opening the Black Box: Automated Decisions and the GDPR*.
- Mothilal, R. K., Sharma, A. and Tan, C. (2020). *Explaining Machine Learning Classifiers through Diverse Counterfactual Explanations*.
