# Uncapped Strategy Analyst Case Study

This repository contains my solution for the Uncapped case study. The core objective is to segment applicants by credit risk and translate those segments into underwriting and pricing recommendations that improve portfolio profitability relative to a **2.5 percent minimum return target**.

## What I built

### Two linked predictive models
I trained two XGBoost classifiers:

1. **Acceptance model**  
   Predicts the probability an application is accepted.

2. **Default model (selection bias corrected)**  
   Predicts probability of default for issued loans, but is trained with **inverse propensity weights** derived from the acceptance model. This addresses the fact that default outcomes are only observed for accepted loans (rejected applications never receive a loan, so their default outcome is unobserved).

In the notebook, the default model uses weights:

- `w = 1 / clip(p_accept, 0.05, 1.0)`

so accepted loans that resemble rarely accepted profiles contribute more to learning (each such loan proxies many similar rejected applicants).

### Risk band segmentation
After predicting a default probability for every application, I map each applicant into one of four fixed risk bands:

- **Band A:** PD < 10%  
- **Band B:** 10% to 20%  
- **Band C:** 20% to 50%  
- **Band D:** PD ≥ 50%

I then produce a band level summary of:
- Business profile and loan request characteristics
- Current underwriting behavior (acceptance rate, fees)
- Risk and return performance

## Key findings (high level)

- Observed credit losses are concentrated in the highest risk segment (Band D), while Bands A to C have had no defaults to date in the provided history.
- Current acceptance rates are not monotonic with modeled risk (Band D is accepted at roughly the same rate as Bands A and B, while Band C is treated more conservatively than Band D). 
- Portfolio performance before recommendations is below target (realized global return is about **0.6 percent** versus a **2.5 percent** minimum target).

## Recommendations (summary)

1. **Align underwriting decisions with risk bands**  
   Embed the default probability model into underwriting, assign each application to Bands A to D, and scale decision stringency with risk.

2. **Avoid standard approvals in Band D**  
   Because Band D concentrates losses and is strongly value destructive, decline Band D under normal policy.

3. **Focus core lending on Bands A and B, treat Band C as optional growth**  
   Bands A and B provide profitable volume, with Band B acting as the main profit workhorse. Band C can be added later with limits and monitoring depending on risk appetite.

4. **Pricing by band**  
   Maintain risk based pricing, but ensure the top end is meaningfully differentiated (Band D should not be priced similarly to Band C given the risk).

## Files

- **Notebook:** `Uncapped_case_study.ipynb`  
  End to end pipeline: data cleaning, feature engineering, modeling, segmentation, and recommendation calculations.

- **Report:** `Uncapped (Case Study Report).docx`  
  Written explanation of methodology, segment insights, and recommendations.

## Outputs

Running the notebook writes:

- `df_complete.csv`  
  The enriched application level dataset with predicted default probability and assigned risk band.

- `summary_report.csv`  
  Band level summary produced via DuckDB.

## How to run

### 1. Put the input file in the working directory
The notebook expects:

- `Uncapped_Strategy_Analyst_Case_Data.xlsx`

### 2. Create an environment and install dependencies
```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# Mac or Linux:
source .venv/bin/activate

pip install -U pip
pip install pandas numpy matplotlib scikit-learn xgboost lightgbm catboost optuna duckdb openpyxl
```

### 3. Run the notebook
```bash
jupyter lab
```
Open `Uncapped_case_study.ipynb` and run cells top to bottom.

## Notes on modeling choices

- Hyperparameter tuning is performed with Optuna.
- Decision thresholds are tuned (F1 based) and probabilities are calibrated (isotonic in the notebook).
- The selection bias correction uses acceptance probability weighting to better reflect the full applicant pipeline in default risk estimation.
