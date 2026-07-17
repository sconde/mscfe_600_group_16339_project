# GWP1 - Review & Critique Log

A rigorous review of the GWP1 submission: correctness, methodology, rubric fit, completeness,
and voice. Findings are listed by severity with the action taken. The voice target is the group's
own academic style, modeled on Conde, Fekete, and Shadid, arXiv 1806.08693 (collective "we/our",
active voice, cautious-but-confident, precise, plain ASCII prose).

## Severity legend
- **[Integrity]** a number or claim that must be verifiable
- **[Method]** a modeling issue that affects how results should be read
- **[Rubric]** a gap against the grading rubric
- **[Complete]** a missing piece of an assignment question
- **[Style]** voice or formatting

---

## Findings and actions

### [Integrity] Maximum drawdown was not computed in the notebook
- **Where:** `report/part1-answers.md`, Q2 ("maximum drawdown is -60.2%").
- **Issue:** every other Q2 number matched the notebook, but the drawdown figure was not produced
  anywhere in the code, so it was unverifiable.
- **Action:** added a max-drawdown computation to the notebook (Section A.3, peak-to-trough on the
  adjusted-close path). It prints **-60.2%**, which matches the figure in the report. The number
  is now notebook-sourced. **Resolved.**

### [Method] The ~80% accuracy was inflated by an overnight-gap artifact
- **Where:** `report/part1-answers.md`, Step 3; notebook Section A.
- **Issue:** the target is the next-day **Open** direction, and the next Open is nearly equal to
  the current Close. The top-ranked indicators (Balance of Power, Increasing/Decreasing flags,
  daily return) are all functions of Close minus Open, so they nearly **encode** the label rather
  than forecast it. An 80% two-indicator accuracy for next-day direction is implausible on its
  face, and this mechanism explains it.
- **Action:** added a **robustness check** to the notebook (Section A.9) that re-runs the identical
  pipeline on a cleaner next-day **Close-to-Close** target. Result: accuracy falls from **80.12%**
  to **54.22%** at two indicators, and to roughly 53% across the small subsets, against a 49.3%
  base rate. Step 3 of the report now names the mechanism, reports the collapse, and clarifies that
  the paper's feature-selection conclusion still holds even though the accuracy level does not.
  **Resolved and turned into a strengthening finding.**

### [Method] Cross-validation is not time-aware
- **Where:** notebook Section A.7; Step 3.
- **Issue:** stratified k-fold on ordered price data allows future information into training folds.
  This is faithful to the paper but is not a walk-forward design.
- **Action:** Step 3's recommendation now explicitly calls for a time-aware (walk-forward)
  validation before any real use. **Documented; left faithful to the paper by design.**

### [Rubric] No non-technical (investor-facing) report
- **Where:** whole report; the rubric awards 40 points for technical **and** non-technical reports.
- **Issue:** the draft was entirely question-and-answer and technical; there was no plain-language
  investor section.
- **Action:** added `report/non-technical-summary.md` - a short investor-facing summary with the
  three parts the rubric asks for (clear explanation of results, recommended action, factors that
  affect the position) and **no** model, algorithm, or code names. **Resolved.**

### [Rubric] "Names of Python code" appeared in technical prose
- **Where:** `report/part1-answers.md`, Q1.
- **Issue:** the rubric says the technical report should exclude the names of Python code; the
  prose named the "Pandas-TA" library.
- **Action:** replaced with "a technical-analysis library" and spelled out "On-Balance Volume"
  instead of the abbreviation. **Resolved.**

### [Complete] Q5 did not give the Jaccard formula
- **Where:** `report/part1-answers.md`, Q5.
- **Issue:** the assignment asks "what is the Jaccard distance?"; the answer compared it to other
  metrics but never stated the definition, range, or the paper's ETF values.
- **Action:** added the definition (symmetric difference over union, equal to one minus
  intersection-over-union), the 0-to-1 range, and the reported distances J(ECH,EWZ)=0.33,
  J(ECH,IVV)=0.64, J(EWZ,IVV)=0.53. **Resolved.**

### [Complete] Q3 did not give an explicit Section 3 outline
- **Where:** `report/part1-answers.md`, Q3.
- **Issue:** the assignment asks to "outline the new section 3 with subcategories"; the answer
  described it in prose only.
- **Action:** added a five-item bulleted outline (filter selectors, model-based selectors,
  consensus selection, predictive model, validation). **Resolved.**

### [Complete] Part 2 was under the ~5-page expectation
- **Where:** `report/part2-user-guide.md`.
- **Issue:** the draft was roughly two pages of prose; the assignment expects about five.
- **Action:** expanded all seven sections with concrete detail (specific sources and datasets,
  time-zone/look-ahead handling, manipulation and sampling risks, privacy and market-integrity
  ethics, a fuller literature discussion) while keeping the collective voice and the code block.
  **Resolved.**

### [Style] Non-ASCII punctuation and math symbols introduced during editing
- **Where:** `report/part1-answers.md` (Q3 outline, Q5, Step 3).
- **Issue:** em-dashes and the set symbols for symmetric difference, union, and approximately-equal
  do not match the group's plain-ASCII voice and can produce paste artifacts in a word processor.
- **Action:** converted em-dashes to colons/commas and rewrote the Jaccard definition in words.
  A scan of `report/` now shows zero em-dashes, en-dashes, set symbols, or `â`/`Â` artifacts.
  **Resolved.**

### [Citations] Sun et al. metadata
- **Where:** `report/references.md`.
- **Check:** verified against Crossref (DOI 10.1186/s40854-024-00652-0). Authors
  (Yunchuan Sun; Lu Liu; Ying Xu; Xiaoping Zeng; Yufeng Shi; Haifeng Hu; Jie Jiang; Ajith
  Abraham), *Financial Innovation*, vol. 10, article 127, 2024 - all correct as written.
  **No change needed.**

---

## What we checked and found correct (left unchanged)
- Q1-Q4 substance and the descriptive-vs-model distinction.
- Every Q2 statistic (annualized volatility 20.8%, cumulative return -30.1%, mean 35.30, median
  33.96) matches notebook output.
- The Step 3 accuracy table (80.12% at two indicators down to 74.70% for all 21) matches the
  notebook exactly.
- The collective "we/our" voice, active constructions, and honest hedging already present in the
  drafts - consistent with the arXiv 1806.08693 style model.
- MLA Works Cited structure and the in-text citation map.

## Verification performed
- Rebuilt and re-executed `notebooks/GWP1_code.ipynb` end-to-end (Python 3.12, numpy<2):
  **0 errored cells**; drawdown, both accuracy tables, and all four figures regenerated.
- Cross-checked every number in `report/*.md` against the fresh notebook outputs.
- Scanned `report/` for encoding artifacts: none.

## Still the group's to do (not automatable)
- Read every paragraph aloud and adjust any wording that does not sound like you; the prose is a
  strong scaffold in your voice, but Turnitin and the WQU AI policy require it to be truly yours.
- Enforce the strict one-page limit on Q2 and the ~five-page target on Part 2 in the Google Doc.
- Fill Report Template page 1 (names, emails, contribution) and designate the submitter.
- Re-run the notebook in Colab and export the output PDF; upload the report PDF separately.
