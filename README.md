# Credit Default Risk Scorecard

A machine learning project that predicts the probability of a loan applicant defaulting, and turns it into an interpretable credit scorecard with approve / review / reject decisions.

Built on the German Credit (Statlog) dataset — 1,000 applicants, 30% defaults.

## What it does

- Cleans and explores the data
- Transforms features using Weight of Evidence (WoE) and ranks them by Information Value (IV)
- Trains a calibrated logistic regression with balanced class weights
- Chooses the decision cut-off from a cost rule (a missed default costs 5× a false alarm)
- Converts scores into a points-based scorecard with risk bands
- Runs a basic fair-lending audit

## Results

- AUC: 0.809
- Gini: 0.619
- KS: 0.514
- PSI (stability): 0.043

Moving the cut-off from 0.50 to 0.25 raised the share of true defaulters caught from 58% to 81%, while keeping costs low.

## Tools

Python · pandas · scikit-learn · matplotlib

## Dataset

UCI Statlog (German Credit Data), Hofmann, H. (1994). Licensed CC BY 4.0.

## Author

C. Shyam Sunder — M.Sc. Statistics
Built as a learning project. Feedback welcome.
