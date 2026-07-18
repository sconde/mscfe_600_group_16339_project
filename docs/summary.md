# GWP1 - Assignment Summary and Rubric Map

**Course:** MScFE 600 Financial Data
**Project:** Group Work Project 1, Module 6
**Group:** 16339
**Members:** Sidafa Conde, Lizzie Luhanga, Saurav Pal
**Deadline:** July 21, 2026, 8:00 PM EDT

## Project Scope

The project has two required parts because the group has three members.

Part 1 reviews Sagaceta-Mejia, Sanchez-Gutierrez, and Fresan-Figueroa (2024). The paper predicts ETF movement direction using daily market data, engineered technical indicators, feature selection, and a multilayer perceptron classifier. Our report answers Q1-Q5, explains the financial problem, discusses the application, and replicates a simplified ECH experiment using Pearson correlation.

Part 2 is a user guide for social-media sentiment as an alternative dataset. The guide covers sources, data types, quality issues, ethics, import structure, exploratory analysis, and supporting literature.

## Locked Decisions

| Decision | Choice | Reason |
|---|---|---|
| ETF | ECH | Emerging-market ETF, available data, aligned with the paper |
| Replication filter | Pearson correlation | Transparent and easy to explain |
| Part 2 dataset type | Social-media sentiment | Fits Sun et al.'s behavioral alternative-data category |
| Notebook sample | Illustrative sentiment sample | Demonstrates workflow without API keys |

## Current Quantitative Results

| Result | Value |
|---|---:|
| ECH cleaned observations | 2,487 |
| Engineered indicators | 21 |
| Share of up days | 49.1% |
| Annualized volatility | 20.8% |
| Cumulative return, 2010-2019 | -30.1% |
| Maximum drawdown | -60.2% |
| Best Open-to-Open median 10-fold accuracy | 80.12% with 2 indicators |
| Full-set Open-to-Open median 10-fold accuracy | 74.70% with 21 indicators |
| Cleaner Close-to-Close robustness result | 54.22% with 2 indicators; near 53% across small subsets |

The high Open-to-Open accuracy is included for comparability with the paper-style target. The
robustness check is the defensible conclusion: once the target is changed to next-day
Close-to-Close direction, the signal falls close to the no-skill line.

## Rubric Map

| Rubric component | Where addressed |
|---|---|
| Quantitative analysis | Part 1 Q1-Q5, ECH replication, accuracy table, Part 2 EDA discussion |
| Technical report | Step 3 replication: data, method, results, interpretation, recommendation |
| Non-technical report | Step 1 and Step 2: financial problem, practical action, portfolio relevance |
| Writing and formatting | Clean group voice, generated figures, MLA Works Cited |

## Final Assembly Notes

Use the WQU report template for the submitted PDF. Fill in all emails, contribution flags, and the submitting member. Insert the figures from `figures/` with readable captions and labelled axes. Export the report as a PDF and upload it separately from the notebook zip.
