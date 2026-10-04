# Explainable Credit Risk

## SHAP, LIME, and Counterfactual Explanations for Black-Box Credit Models

Machine learning models can achieve strong predictive performance in financial applications, but increasing model complexity often comes at the cost of interpretability.

In credit risk, this creates an important problem: if a model classifies an applicant as high risk, it is not enough to simply say that the model predicts default. Lenders, regulators, analysts, and customers may also want to understand **why the model made that prediction and whether the explanation itself can be trusted**.

This project investigates three widely used post-hoc explainability methods:

- **SHAP**
- **LIME**
- **Counterfactual explanations**

Rather than treating these methods as interchangeable, the project asks what each method actually explains and where each one can fail.

---

## Research Question

> How well can SHAP, LIME, and counterfactual explanations explain the predictions of a black-box credit-risk model, and what limitations remain?

The central idea of the project is that these methods answer different questions:

| Method | Main Question |
| --- | --- |
| SHAP | Which features contributed to this prediction? |
| LIME | What simple model approximates the black box around this applicant? |
| Counterfactuals | What changes would cause the model's prediction to change? |

These questions are related, but they are not equivalent.

In particular, explaining a model's prediction should not automatically be interpreted as identifying the real-world causes of default.

---

## Dataset

The project uses the **UCI Default of Credit Card Clients dataset**.

The dataset contains information on **30,000 credit-card customers**, including:

- credit limit;
- repayment status;
- monthly bill amounts;
- monthly payment amounts;
- age;
- education;
- marital status;
- demographic variables.

The target variable indicates whether the customer **defaults in the following month**.

For this project, predicted default probability is used to construct a hypothetical credit-risk decision rule. An applicant with sufficiently high predicted default risk can therefore be treated as a hypothetical declined applicant for the explainability analysis.

---

## Methodology

Two classification models are trained on the same data.

### Logistic Regression

Logistic regression is used as an interpretable baseline.

Its relatively simple functional form allows us to examine how input variables affect the estimated probability of default.

### XGBoost

XGBoost is used as the more complex model.

Tree ensembles can capture nonlinearities and interactions that a standard logistic regression may miss, but their predictions are substantially harder to interpret directly.

The models are compared using appropriate classification metrics such as:

- ROC-AUC;
- accuracy;
- precision;
- recall;
- F1 score;
- confusion matrix.

The purpose is not only to determine which model predicts better, but also to examine the trade-off between predictive flexibility and interpretability.

---

## Explainability Experiment

After training the models, the analysis selects **one high-risk applicant from the test set**.

The XGBoost prediction for this same applicant is then explained using SHAP, LIME, and counterfactual explanations.

Using the same observation across all three methods makes it possible to directly compare what each method considers an "explanation."

---

## 1. SHAP

SHAP assigns a contribution to each feature based on ideas from cooperative game theory.

For the selected applicant, a SHAP waterfall plot shows how individual variables move the prediction away from the model's baseline prediction.

A global SHAP beeswarm plot is also used to examine which variables are most influential across the test set.

### Stress Test: Correlated Features

The dataset contains groups of strongly related variables, including monthly bill amounts and repayment histories.

For example:

`BILL_AMT1`, `BILL_AMT2`, ..., `BILL_AMT6`

contain overlapping information about the customer's credit position.

The experiment examines how SHAP distributes attribution among correlated predictors.

The objective is not to show that SHAP is mathematically incorrect. Instead, it illustrates that **individual feature attribution can become ambiguous when multiple predictors contain similar information**.

---

## 2. LIME

LIME explains an individual prediction by generating observations around the selected applicant, obtaining predictions from the black-box model, and fitting a simpler local surrogate model.

The resulting local feature importance is compared with the SHAP explanation for the same applicant.

### Stress Test: Stability

LIME is run multiple times using different random seeds.

Because the method relies on sampling a synthetic local neighborhood, its explanation can change even when:

- the applicant is unchanged;
- the XGBoost model is unchanged;
- the predicted default probability is unchanged.

This experiment evaluates the stability of local explanations.

---

## 3. Counterfactual Explanations

Counterfactual explanations ask:

> What would need to be different for this applicant to receive a different model prediction?

Counterfactuals are generated using **DiCE**.

The experiment first allows the algorithm to search relatively freely for changes that move the applicant from high predicted default risk to lower predicted risk.

### Stress Test: Feasibility

An unconstrained counterfactual may recommend unrealistic changes, such as modifying age or other characteristics that cannot reasonably be changed by the applicant.

The experiment therefore compares:

1. **Unconstrained counterfactuals**
2. **Constrained counterfactuals**, where immutable or unrealistic changes are restricted

This demonstrates that a mathematically valid counterfactual is not necessarily an actionable financial recommendation.

---

## Main Hypothesis

The project does not assume that one explainability method is universally superior.

Instead, the hypothesis is that:

> **SHAP, LIME, and counterfactual explanations provide different views of model behavior, but none of them fully turns a black-box model into an inherently interpretable model.**

In particular:

- SHAP provides feature attribution;
- LIME provides a local approximation;
- counterfactuals describe changes that cross a model decision boundary.

None of these methods, by itself, establishes causality.

---

## Key Distinction

A central theme of the project is the difference between:

**Prediction**

> What outcome does the model predict?

**Attribution**

> Which inputs are associated with the model's prediction?

**Counterfactual explanation**

> Which input changes would change the model's decision?

**Causality**

> Which real-world intervention would actually change the outcome?

These are not the same question.

---

## Repository Structure

```text
explainable-credit-risk/
│
├── README.md
├── requirements.txt
│
├── data/
│   └── README.md
│
├── notebooks/
│   └── 01_explainable_credit_risk.ipynb
│
├── src/
│   ├── preprocessing.py
│   ├── models.py
│   └── explainers.py
│
├── figures/
│   ├── model_performance.png
│   ├── shap_waterfall.png
│   ├── shap_beeswarm.png
│   ├── lime_comparison.png
│   ├── lime_stability.png
│   └── counterfactual_comparison.png
│
└── presentation/
    └── explainable_credit_risk.pptx
```

---

## Technologies

The project is implemented in Python using libraries including:

```text
pandas
numpy
scikit-learn
xgboost
shap
lime
dice-ml
matplotlib
```

Exact package versions are provided in `requirements.txt`.

---

## Reproducibility

Random seeds are fixed where possible so that model training, train/test splitting, and explanation experiments can be reproduced.

The main notebook contains the complete experimental workflow:

1. load and inspect the data;
2. preprocess the variables;
3. split the data into training and test sets;
4. train logistic regression;
5. train XGBoost;
6. compare predictive performance;
7. select one high-risk applicant;
8. generate SHAP explanations;
9. generate and stress-test LIME explanations;
10. generate unconstrained and constrained counterfactuals;
11. compare the three approaches.

---

## Project Takeaway

Explainability tools can provide valuable information about how complex machine-learning models behave.

However, an explanation should not automatically be interpreted as proof that a model is fair, causal, stable, or trustworthy.

The project therefore treats explainability methods as **diagnostic tools rather than certificates of trust**.

---

## References

Hastie, T., Tibshirani, R., & Friedman, J.  
*The Elements of Statistical Learning: Data Mining, Inference, and Prediction.*

Huyen, C.  
*Designing Machine Learning Systems.*

Lundberg, S. M., & Lee, S. I.  
*A Unified Approach to Interpreting Model Predictions.*

Ribeiro, M. T., Singh, S., & Guestrin, C.  
*"Why Should I Trust You?": Explaining the Predictions of Any Classifier.*

Wachter, S., Mittelstadt, B., & Russell, C.  
*Counterfactual Explanations Without Opening the Black Box: Automated Decisions and the GDPR.*

UCI Machine Learning Repository.  
*Default of Credit Card Clients Dataset.*
