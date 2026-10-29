# Explainable Credit Risk

**University of Toronto | Master of Mathematical Finance | MMF1922H – Data Science**

Machine learning models can predict whether a customer is likely to default on a loan. But in finance, a prediction alone is not enough. We also need to understand **why a model makes a decision, whether its explanation is reliable, and what its limitations are.**

This project explores how different machine learning models can be interpreted and how Explainable AI (XAI) methods help us understand their predictions.

## Project Objective

Our research addresses a central question:

*As models become more complex, how can we explain their predictions, and how much can we trust those explanations?*

We investigate three credit-risk classification models:

- **Logistic Regression:** A relatively transparent model whose predictions can be explained directly through its coefficients.
- **XGBoost:** A more complex tree-based model, where we examine how different measures of feature importance describe its behavior.
- **Neural Network:** A model for which we apply SHAP, LIME, and counterfactual explanations to understand individual predictions and their limitations.

Rather than focusing only on which model predicts best, our goal is to understand **what each explanation method tells us and what it cannot tell us.**

## Dataset

We use the [UCI Default of Credit Card Clients dataset](https://archive.ics.uci.edu/dataset/350/default+of+credit+card+clients), containing information about 30,000 credit-card customers in Taiwan.

The data include credit limits, repayment history, bill amounts, previous payments, and demographic information. Our objective is to predict whether a customer will default on their next payment.

The project includes a separate 24,000-record dataset configuration for the neural network explainability experiments. Since these experiments use different features and data splits, their predictive results are not treated as a directly controlled comparison with the other models.

## Explainability Methods

| Method | What does it help us understand? |
|---|---|
| **Logistic Regression coefficients** | How each input contributes to the model's prediction |
| **XGBoost feature importance** | Which variables matter most to the model, depending on how importance is measured |
| **SHAP** | Which features push a prediction toward higher or lower risk |
| **LIME** | How a simpler model can approximate an individual prediction locally |
| **DiCE counterfactuals** | What changes to a customer's inputs could produce a different prediction |

SHAP, LIME, and DiCE were implemented on the neural network. The other models were examined using their available interpretation and feature-importance methods.

## Key Findings

Our experiments highlight several important results.

**Interpretability does not always mean simplicity.** Logistic Regression allows us to reconstruct individual predictions directly, but correlated variables and categorical encoding can still complicate their interpretation.

**Feature importance depends on how it is measured.** In XGBoost, the latest repayment status ranks first using *gain*, but eighth using *weight*. Different measures can tell different stories about the same model.

**Different explanation tools do not always agree.** SHAP and LIME identified only two common features among their four leading explanations for the selected neural-network prediction. Repeated LIME explanations also showed variation.

**Changing a prediction is not necessarily a realistic solution.** DiCE successfully generated alternative profiles with lower predicted default risk, but some required unrealistic changes to payment history or financial amounts.

## Conclusion

Our main finding is that **no single explanation method tells the whole story**.

Logistic Regression offers direct interpretability, XGBoost provides useful global importance measures, and SHAP, LIME, and DiCE offer complementary ways to investigate complex model predictions.

However, explanations can depend on the method, vary across runs, or suggest changes that are not feasible in practice.

In financial applications, understanding these limitations is just as important as obtaining accurate predictions.

## Repository Structure

```text
data/           Dataset files
notebooks/      Model training, evaluation and explainability
presentation/   Final group presentation
README.md       Project overview
requirements.txt
```

The `notebooks/` folder contains the Logistic Regression, XGBoost, and two Neural Network notebooks. The neural network notebooks document separate experimental configurations.

## Getting Started

Clone the repository, create a Python environment, and install the required packages:

```bash
python -m pip install -r requirements.txt
```

Launch JupyterLab from the `notebooks/` directory:

```bash
cd notebooks
jupyter lab
```

Run the notebooks independently, using the project environment as the kernel. A clean installation has not been fully verified; see [RESULTS.md](RESULTS.md) for reproducibility details.

## References

Yeh, I. *Default of Credit Card Clients*. UCI Machine Learning Repository.

Lundberg & Lee (2017). *A Unified Approach to Interpreting Model Predictions*.

Ribeiro, Singh & Guestrin (2016). *“Why Should I Trust You?”: Explaining the Predictions of Any Classifier*.

Mothilal, Sharma & Tan (2020). *Explaining Machine Learning Classifiers through Diverse Counterfactual Explanations*.
