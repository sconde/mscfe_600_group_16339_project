# MScFE 600 Financial Data - Group Work Project 1

WorldQuant University | Financial Data (MScFE 600) | Group Work Project 1 (GWP1) | Module 6

**Student Group 16339**

| Group Member | Email | Actively Contributed |
|---|---|---|
| Sidafa Conde | sconde89@gmail.com | Yes |
| Lizzie Luhanga | lizzieluhanga44@gmail.com | Yes |
| Saurav Pal | saurvpal.wor@gmail.com | Yes |

Fill in emails and contribution flags on page 1 of the WQU Report Template before submitting.

## Overview

This is a 3-member group, so both parts of the assignment are required.

- **Part 1 - Assessing Models with Alternative Data.** We review Sagaceta-Mejia, Sanchez-Gutierrez, and Fresan-Figueroa (2024), which predicts ETF direction using technical indicators, feature selection, and a neural-network classifier under 10-fold cross-validation. We answer Q1-Q5, discuss the financial problem and application, and replicate a simplified slice of the study on the iShares MSCI Chile ETF (ECH) using Pearson correlation as the feature filter. The technical answers are paired with a short **non-technical (investor-facing) summary** so the report covers both the technical and non-technical reporting the rubric asks for.
- **Part 2 - Evaluating One Type of Alternative Data.** We prepare a practitioner guide on social-media sentiment data, covering sources, data types, quality, ethics, import structure, EDA, and a short literature review.

Note on the fund list: the assignment writes the funds as "ECH, EQZ, or IVV." EQZ appears to be a typo for EWZ, the iShares MSCI Brazil ETF used in the paper. We use ECH, so the typo does not affect the replication.

## Repository Structure

```text
group-project/
|-- README.md
|-- GROUP_WORK_POLICY.md
|-- docs/
|   |-- summary.md
|   |-- execution-plan.md
|   `-- quality-checklist.md
|-- report/
|   |-- part1-answers.md
|   |-- part2-user-guide.md
|   |-- non-technical-summary.md
|   `-- references.md
|-- notebooks/
|   `-- GWP1_code.ipynb
|-- figures/
|-- data/
|   |-- ECH.csv
|   `-- social_sample.csv
|-- REVIEW.md
`-- *.pdf
```

`report/non-technical-summary.md` is the plain-language investor section. It must be included in
the assembled report (it is what earns the non-technical half of the reporting rubric).
`REVIEW.md` is our internal review log and is not part of the submitted report.

## Current Results to Carry Into the Report

From the executed notebook run:

| Item | Result |
|---|---:|
| ECH observations after cleaning | 2,487 |
| Engineered indicators | 21 |
| Share of up days | 49.1% |
| Annualized volatility | 20.8% |
| Cumulative return, 2010-2019 | -30.1% |
| Maximum drawdown | -60.2% |
| Best Open-to-Open median 10-fold accuracy | 80.12% with 2 indicators |
| Full-set Open-to-Open median 10-fold accuracy | 74.70% with 21 indicators |
| Cleaner Close-to-Close robustness result | 54.22% with 2 indicators; near 53% across small subsets |

The 80.12% result is reported because it follows the paper-style Open-price target, but it should
not be treated as live trading skill. The report explains why the Open-to-Open target is optimistic
and uses the Close-to-Close robustness check as the more honest read on predictive strength.

## How to Run the Code

The notebook is written for Google Colab.

1. Upload `notebooks/GWP1_code.ipynb`.
2. Run all cells.
3. Confirm every cell shows output.
4. Export or print the notebook as a PDF with outputs visible.

To run locally, install the dependencies:

```powershell
pip install "numpy<2" pandas scikit-learn matplotlib seaborn yfinance nltk
```

Then execute the notebook in Jupyter. Figures are written to `figures/`.

## Deliverables

- PDF report built from the WQU Report Template, uploaded separately so Turnitin can run. The report must include Part 1 (Q1-Q5 and Steps 1-3), Part 2 (the user guide), the references, and the non-technical (investor-facing) summary from `report/non-technical-summary.md`.
- Zip file containing `GWP1_code.ipynb` and a PDF export of the notebook with outputs shown.
- Page 1 of the report template must include all group member names, emails, and contribution flags.

**Submitting member:** _designate one_
**Submission deadline:** July 21, 2026, 8:00 PM EDT

## Reference Documents

- `MScFE 600 Financial Data GWP1.pdf` - assignment and grading rubric
- `MScFE_600_Financial Data_Group_Work_Project_Template.pdf` - report template
- `10.1515_econ-2022-0073.pdf` - Sagaceta-Mejia et al. (2024), Part 1 source paper
- `GROUP_WORK_POLICY.md` - WQU group-work policy
