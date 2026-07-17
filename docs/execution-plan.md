# GWP1 - Final Execution Plan

## Owners

| Owner | Primary responsibilities |
|---|---|
| Sidafa Conde | Part 1 Q1-Q3, ECH replication review, final voice pass |
| Saurav Pal | Part 1 Q4-Q5, financial problem, application section |
| Lizzie Luhanga | Part 2 user guide, Works Cited, citation checks |
| Shared | Final template assembly, notebook export, submission verification |

## Remaining Steps

1. Run `notebooks/GWP1_code.ipynb` in Google Colab.
2. Confirm outputs match the report values or update the report table if the live data pull changes.
3. Insert the Part 1 and Part 2 text from `report/` into the WQU report template.
4. Add the ECH price chart, feature-correlation chart, accuracy-vs-features chart, and sentiment EDA figure.
5. Check page limits, especially the one-page ECH security description and the approximate five-page Part 2 guide.
6. Confirm every in-text source appears in `report/references.md`.
7. Export the report PDF separately from the notebook zip.
8. Export the notebook with outputs visible and zip it with the `.ipynb` file.
9. Have one designated member submit and confirm the upload before July 21, 2026 at 8:00 PM EDT.

## Important Limitations to Preserve

The replication is intentionally simplified. It uses 21 engineered indicators rather than the paper's full 216-feature panel. It uses Pearson correlation as the selection filter and next-day Open direction as the target. The Part 2 sentiment sample is illustrative and should not be described as a real trading dataset.
