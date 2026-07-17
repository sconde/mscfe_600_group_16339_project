# GWP1 - Review Resolution Log

All substantive review findings have been addressed in the project files. This file now records
where each issue was resolved and separates the few remaining submission actions that require
group information or a final Colab export.

## Resolved Content and Quality Findings

- [x] Rewrote the main report files in collective group voice.
      See `report/part1-answers.md`, `report/part2-user-guide.md`, and
      `report/non-technical-summary.md`.
- [x] Replaced scaffold language with final report prose.
      No scaffold markers or missing-result markers remain in the report files.
- [x] Filled the ECH replication results.
      The report now includes annualized volatility, cumulative return, maximum drawdown, the
      Open-to-Open accuracy table, and the cleaner Close-to-Close robustness check.
- [x] Addressed the optimistic-target problem.
      Part 1 now explains why the Open-to-Open target can overstate predictive skill and reports
      the cleaner Close-to-Close result from the notebook.
- [x] Added the non-technical investor-facing section.
      `report/non-technical-summary.md` is called out in `README.md` and should be inserted into
      the final assembled report.
- [x] Framed the Part 2 sentiment sample correctly.
      The guide and notebook describe the 24-post sample as illustrative, not as evidence for a
      trading rule.
- [x] Corrected the Sun et al. reference and Works Cited.
      See `report/references.md`.
- [x] Cleaned obvious encoding artifacts from the Markdown report and project docs.
- [x] Confirmed the notebook contains executed outputs for the report's stated results.

## Human-Only Items Before Submission

- [ ] Read every paragraph aloud as a group and make any final wording changes needed for the
      prose to sound like the group's own voice.
- [ ] Enforce page limits in the assembled WQU template: Q2 must stay within one page and Part 2
      should be about five pages.
- [ ] Fill Report Template page 1 with all three member emails and contribution flags.
- [ ] Designate the submitting member.
- [ ] Run `notebooks/GWP1_code.ipynb` in Google Colab, confirm every cell shows output, and export
      the notebook as a PDF with outputs visible.
- [ ] Upload the report PDF separately from the zipped notebook so Turnitin can run.
