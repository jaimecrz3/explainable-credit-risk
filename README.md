# Explainable Credit Risk

What do intrinsic interpretation, global feature importance and post-hoc explanations reveal about credit-risk predictions, and where do their usefulness and reliability break down?

This four-person university project studies logistic regression, XGBoost and neural networks using credit-card client data. One member developed logistic regression and its interpretation, one developed XGBoost and its importance analysis, and two developed the neural-network modeling and explanation studies. Predictive performance provides context; the main objective is understanding what each interpretation or explanation answers and what it cannot establish.

## Contributions across the three model families

| Model | Completed interpretation / explanation work | Empirical contribution |
| --- | --- | --- |
| Logistic regression | Coefficients, odds ratios, global interpretation, correlations and exact local prediction decomposition | PAY_0 has coefficient 0.661 and exp(coefficient) 1.937 per standard deviation; a local prediction is reconstructed as 0.9927. Correlated inputs, rare categories and full one-hot coding limit isolated coefficient interpretations. This is intrinsic interpretability, not SHAP. |
| XGBoost | Global importance by gain, weight and cover; ranking comparison and gain plot | PAY_0 ranks 1 / 8 / 3 under the three definitions; BILL_AMT1 ranks 10 / 1 / 11. Importance depends on its definition and gives neither direction nor an individual attribution. SHAP, LIME and DiCE are not implemented for this model. |
| Neural network | Original modeling notebook plus a separate complete SHAP/LIME/DiCE study | Local/global attribution, surrogate explanations, repeated-run stability, method agreement and constrained/unrestricted counterfactuals; the explanation model uses Configuration B below. |

The project compares **types of information**, not explanations of one common model or applicant across all notebooks. Within Configuration B, SHAP, LIME and DiCE do examine the same network and selected applicant.

| Method | Question | Limitation explored |
| --- | --- | --- |
| SHAP | Which inputs contribute relative to a reference population? | Estimation stability; dependence on background and correlated inputs |
| LIME | Which simple surrogate approximates the model around one case? | Variation across sampled neighborhoods |
| Counterfactuals (DiCE) | Which input changes cross the decision boundary? | Model validity versus real-world feasibility |

Attribution, local approximation and decision-boundary changes are different objects. None establishes causality, fairness or trustworthy deployment by itself.

## Two separate experimental configurations

| Setting | A: common predictive comparison | B: neural-network explainability |
| --- | --- | --- |
| Input | Excel, 30,000 rows | PyCaret CSV, 24,000 rows |
| Predictors | 23; drop ID and target | 21; also exclude SEX and MARRIAGE |
| Split | Stratified 80/20, seed 42 | Stratified 75/25, seed 42 |
| Train / test rows | 24,000 / 6,000 | 18,000 / 6,000 |
| Categorical inputs | SEX, EDUCATION, MARRIAGE | EDUCATION |
| Models | Logistic regression, MLP, XGBoost | Separate MLP |
| Decision rule | Default probability >= 0.50 | Default probability >= 0.50 |

Both configurations standardize numeric inputs and one-hot encode the listed categorical inputs, retaining undocumented codes. Preprocessing is fitted on the outer training set. The same seed does not make different input populations share a test set. **Metrics from A and B are not directly comparable.** Excluding demographic inputs in B does not establish fairness.

The logistic baseline uses `LogisticRegression(max_iter=1000, random_state=42)` with existing defaults. Both MLPs use `(64, 32)` hidden layers, ReLU, Adam, alpha 0.0001, initial learning rate 0.001, batch size 256, at most 300 epochs, validation fraction 0.15 and early-stopping patience 20. XGBoost uses depth 4, minimum child weight 5, at most 2,000 rounds, learning rate 0.03, row/column subsampling 0.8, lambda 1.0, histogram trees, log-loss and early-stopping patience 50 on a stratified 15% inner validation split. The notebooks contain the complete preserved estimators.

## Implemented post-hoc explanation experiments

Configuration A compares predictive performance, explains logistic coefficients and reconstructs one prediction algebraically. XGBoost includes built-in importance, correlations, a raw-input prediction wrapper and applicant selection. It **does not execute SHAP, LIME or DiCE**.

Configuration B applies all three methods to the same network and applicant (saved CSV index 15110, default probability 0.685):

- Kernel SHAP: 100 real background rows; local waterfall with 2,000 samples; five-seed local stability experiment; global summary on 200 test cases with 500 samples each.
- LIME: 5,000 samples per explanation; 20 seeds; top-four overlap and conditional weight summaries.
- DiCE random search: five free and five constrained counterfactuals. The constrained search varies PAY_0 and BILL_AMT1 downward and PAY_AMT1/PAY_AMT2 upward, with payment caps based on training percentiles.

Saved outputs show SHAP top-four overlap of 4/4 against the first run, mean pairwise LIME overlap of 2.61/4, and SHAP/LIME rank overlap of 2/4. These use different comparison definitions and cover one applicant. All ten saved counterfactuals cross the threshold; this does not demonstrate actionable financial advice.

The proposed SHAP correlated-feature attribution stress test is not implemented. Correlation matrices and repeated sampling seeds do not substitute for it. Historical XGBoost slide claims retained in notebook 04 are explicitly unverified; the source slides/model are absent.

See [results and reproducibility](RESULTS.md) for separate metric tables, provenance and scientific limitations, and the [data dictionary](data/README.md) for dataset details.

## Repository and reading order

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
    project_presentation.pptx
    presentation_script.md
    speaker_cheatsheet.md
```

Each notebook is self-contained. Read 01-03 for Configuration A and 04 for the complete neural-network explanation experiment in B. Notebooks 02 and 04 remain separate because their datasets, predictors and splits differ; attaching the explanations in 04 to the fitted model in 02 would misrepresent the experiment. Their repeated modeling code is preserved intentionally. PyCaret identifies the CSV source, not a modeling framework used by this project.

There is no required combined notebook, shared runtime or serialized trained model. Plots are embedded; running XGBoost also creates an optional `figures/` directory with two PNGs.

## Group presentation

- [Editable PowerPoint](presentation/project_presentation.pptx): ten slides with four consecutive speaking sections.
- [Complete English script](presentation/presentation_script.md): slide timings and handovers; each speaker has a scheduled 2:30, totaling 10:00.
- [Speaker cheatsheet](presentation/speaker_cheatsheet.md): individual rehearsal cues and likely questions.

Speaker 1 presents the question and logistic interpretation (slides 1-3); Speaker 2 presents XGBoost importance (4-5); Speaker 3 presents the neural-network context and SHAP (6-7); Speaker 4 presents LIME, stability, DiCE and the shared conclusions (8-10). Delivery time depends on speaking pace and pauses; the schedule is a rehearsal target. Speaker notes reproduce the spoken script. The presentation uses recorded evidence and does not add experiments.

## Run locally

The checked local environment uses Python 3.14.7. `requirements.txt` records installed direct dependencies, not a recovered historical environment or complete transitive lockfile. A fresh installation has not been verified; see `RESULTS.md` for actual execution checks.

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m jupyterlab
```

Open each notebook under `notebooks/`, choose the environment's kernel and restart/run all independently. **The kernel working directory must be `notebooks/`** because data paths are relative to it. Datasets are included; downloading is unnecessary. Notebook 04 retains its download fallback when the CSV is absent; an upstream download is not guaranteed to match the included file.

To preserve historical evidence, execute a copy or do not save over the originals. Notebook 01 retains an optional `%pip install -q xlrd` cell, unnecessary after installing requirements. The final external-slide comparison cell in notebook 04 is historical and has no recorded output; it is not validated cross-model evidence.

## Interpretation limits

These historical client observations do not constitute a validated application-underwriting system. The threshold creates a hypothetical decline decision. Test-set threshold tables are descriptive; operational selection needs validation and explicit costs. Inner early-stopping validation sees preprocessing fitted on the full outer training partition; this existing limitation is documented and preserved. Calibration has not been established.

Correlation, ambiguous repayment codes, single-case selection and synthetic neighborhoods limit interpretation. Search failure does not prove counterfactual impossibility, and changing historical inputs is not an established causal intervention. No temporal validation, fairness audit or uncertainty analysis supports deployment claims.

## References

- UCI Machine Learning Repository. *Default of Credit Card Clients*, dataset 350. Source links: `data/README.md`.
- Lundberg and Lee (2017). *A Unified Approach to Interpreting Model Predictions*.
- Ribeiro, Singh and Guestrin (2016). *Why Should I Trust You?: Explaining the Predictions of Any Classifier*.
- Wachter, Mittelstadt and Russell (2017). *Counterfactual Explanations Without Opening the Black Box: Automated Decisions and the GDPR*.
- Mothilal, Sharma and Tan (2020). *Explaining Machine Learning Classifiers through Diverse Counterfactual Explanations*.
- Hastie, Tibshirani and Friedman. *The Elements of Statistical Learning*.
- Huyen. *Designing Machine Learning Systems*.
